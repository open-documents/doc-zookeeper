
参考文档：https://zookeeper.apache.org/doc/r3.9.0/javaExample.html


# Executor

```java
public class Executor implements Watcher, Runnable, DataMonitor.DataMonitorListener{  
    String znode;  
    DataMonitor dm;  
    ZooKeeper zk;  
    String filename;  
    String exec[];  
    Process child;  
  
    public Executor(String hostPort, String znode, String filename,  
                    String exec[]) throws KeeperException, IOException {  
        this.filename = filename;  
        this.exec = exec;  
        zk = new ZooKeeper(hostPort, 3000, this);  
        dm = new DataMonitor(zk, znode, null, this);  
    }  
  
  
    public static void main(String[] args) {  
        if (args.length < 4) {  
            System.err.println("USAGE: Executor hostPort znode filename program [args ...]");  
            System.exit(2);  
        }  
        String hostPort = args[0];  
        String znode = args[1];  
        String filename = args[2];  
        String exec[] = new String[args.length - 3];  
        System.arraycopy(args, 3, exec, 0, exec.length);  
        try {  
            new Executor(hostPort, znode, filename, exec).run();  
        } catch (Exception e) {  
            e.printStackTrace();  
        }  
    }  
    public void process(WatchedEvent event) {  
        dm.process(event);  
    }  
    public void run() {  
        try {  
            synchronized (this) {  
                while (!dm.dead) {  
                    wait();  
                }  
            }  
        } catch (InterruptedException e) {  
        }  
    }  
  
    public void closing(int rc) {  
        synchronized (this) {  
            notifyAll();  
        }  
    }  
  
    static class StreamWriter extends Thread {  
        OutputStream os;  
  
        InputStream is;  
  
        StreamWriter(InputStream is, OutputStream os) {  
            this.is = is;  
            this.os = os;  
            start();  
        }  
  
        public void run() {  
            byte b[] = new byte[80];  
            int rc;  
            try {  
                while ((rc = is.read(b)) > 0) {  
                    os.write(b, 0, rc);  
                }  
            }  
            catch (IOException e) {  
            }  
  
        }  
    }  
  
    public void exists(byte[] data) {  
        if (data == null) {  
            if (child != null) {  
                System.out.println("Killing process");  
                child.destroy();  
                try {  
                    child.waitFor();  
                } catch (InterruptedException e) {  
                }  
            }  
            child = null;  
        } else {  
            if (child != null) {  
                System.out.println("Stopping child");  
                child.destroy();  
                try {  
                    child.waitFor();  
                } catch (InterruptedException e) {  
                    e.printStackTrace();  
                }  
            }  
            try {  
                FileOutputStream fos = new FileOutputStream(filename);  
                fos.write(data);  
                fos.close();  
            } catch (IOException e) {  
                e.printStackTrace();  
            }  
            try {  
                System.out.println("Starting child");  
                child = Runtime.getRuntime().exec(exec);  
                new StreamWriter(child.getInputStream(), System.out);  
                new StreamWriter(child.getErrorStream(), System.err);  
            } catch (IOException e) {  
                e.printStackTrace();  
            }  
        }  
    }  
}
```

# DataMonitor

```java
public class DataMonitor implements Watcher, AsyncCallback.StatCallback {  
  
    ZooKeeper zk;  
    String znode;  
    Watcher chainedWatcher;  
    boolean dead;  
    DataMonitorListener listener;  
    byte prevData[];  
  
    public DataMonitor(ZooKeeper zk, String znode, Watcher chainedWatcher, 
    DataMonitorListener listener) {  
        this.zk = zk;  
        this.znode = znode;  
        this.chainedWatcher = chainedWatcher;  
        this.listener = listener;  
        // Get things started by checking if the node exists. We are going to be completely event driven
        // 向服务端注册默认Watcher, 以使服务端的this.znode发生变化时, 客户端能够监听到事件
        zk.exists(znode, true, this, null);  
    }  
  
    /**  
     * Other classes use the DataMonitor by implementing this method     */    
    public interface DataMonitorListener {  
        /**  
         * The existence status of the node has changed.         */        void exists(byte data[]);  
  
        /**  
         * The ZooKeeper session is no longer valid.         *         * @param rc  
         *                the ZooKeeper reason code  
         */        void closing(int rc);  
    }  
  
    public void process(WatchedEvent event) {  
        String path = event.getPath();  
        if (event.getType() == Event.EventType.None) {  
            // We are are being told that the state of the connection has changed  
            switch (event.getState()) {  
                case SyncConnected:  
                    // In this particular example we don't need to do anything  
                    // here - watches are automatically re-registered with                    // server and any watches triggered while the client was                    // disconnected will be delivered (in order of course)                    break;  
                case Expired:  
                    // It's all over  
                    dead = true;  
                    listener.closing(KeeperException.Code.SessionExpired);  
                    break;            }  
        } else {  
            if (path != null && path.equals(znode)) {  
                // Something has changed on the node, let's find out  
                zk.exists(znode, true, this, null);  
            }  
        }  
        if (chainedWatcher != null) {  
            chainedWatcher.process(event);  
        }  
    }  
  
    public void processResult(int rc, String path, Object ctx, Stat stat) {  
        boolean exists;  
        switch (rc) {  
            case KeeperException.Code.Ok:  
                exists = true;  
                break;            case KeeperException.Code.NoNode:  
                exists = false;  
                break;            case KeeperException.Code.SessionExpired:  
            case KeeperException.Code.NoAuth:  
                dead = true;  
                listener.closing(rc);  
                return;            default:  
                // Retry errors  
                zk.exists(znode, true, this, null);  
                return;        }  
  
        byte b[] = null;  
        if (exists) {  
            try {  
                b = zk.getData(znode, false, null);  
            } catch (KeeperException e) {  
                // We don't need to worry about recovering now. The watch  
                // callbacks will kick off any exception handling                e.printStackTrace();  
            } catch (InterruptedException e) {  
                return;  
            }  
        }  
        if ((b == null && b != prevData)  
                || (b != null && !Arrays.equals(prevData, b))) {  
            listener.exists(b);  
            prevData = b;  
        }  
    }  
}
```