# AnrWatcher
Tool for anr watch.
a copy version from [here](https://gist.github.com/dodola/7677cdbd0759a9d603b4)

```Java
import android.os.Debug;
import android.os.Handler;
import android.os.Looper;

import java.io.File;
import java.io.FileWriter;
import java.io.IOException;
import java.text.SimpleDateFormat;
import java.util.Date;

import dodola.watcher.config.WatcherConfig;
import dodola.watcher.utils.FileUtil;


public class AnrWatcher extends Thread {
    private final String LOG_NAME_START = "anr_trace_";
    private volatile int prevTimeTick;
    private volatile int nextTimeTick;
    /**
     * 主线程Handler,
     */
    private Handler mainUIHandler;
    /**
     *
     */
    private Runnable prevTimeTickRunnable = new Runnable() {
        @Override
        public void run() {
            prevTimeTick = (prevTimeTick + 1) % 100;
        }
    };
    /**
     * 
     */
    private Runnable nextTimeTickRunnable = new Runnable() {
        @Override
        public void run() {
            nextTimeTick = (nextTimeTick + 1) % 100;
        }
    };

    /**
     * 判定anr开始的时限,在主线程卡住5000毫秒的时候,开始记录method_trace
     */
    private final int DEFAULT_ANR_TIME = 5000;

    public AnrWatcher() {
        setName("DODO_ANR_WATCHER");
        mainUIHandler = new Handler(Looper.getMainLooper());
    }

    public void run() {
        String name = "";
        boolean mayAnr = false;
        while (true) {
            int lastPrevTick = this.prevTimeTick;
            int lastNextTick = this.nextTimeTick;
            this.mainUIHandler.post(this.prevTimeTickRunnable);
            this.mainUIHandler.post(this.nextTimeTickRunnable);
            try {
                Thread.sleep(50);
                if (this.prevTimeTick == lastPrevTick) {
                    name = new SimpleDateFormat("MM_dd_HH_mm_ss").format(new Date());
                    Debug.startMethodTracing(WatcherConfig.getWatcherLogDir() + LOG_NAME_START + name);
                    mayAnr = true;
                }
                if (mayAnr) {
                    mayAnr = false;
                    try {
                        Thread.sleep(DEFAULT_ANR_TIME);
                        if (this.nextTimeTick == lastNextTick) {
                            Debug.stopMethodTracing();
                            return;
                        }
                        Debug.stopMethodTracing();
                        FileUtil.deleteFile(WatcherConfig.getWatcherLogDir() + LOG_NAME_START + name + ".trace");
                        name = "";
                    } catch (InterruptedException e2) {
                        return;
                    }
                }
            } catch (InterruptedException e3) {
                return;
            }
        }
    }
}
```
