# Java定时任务

## 背景

在现实的项目中我们通常有一些需要定时处理的任务。

例如，定时向客户发送一些通知邮件。

定时进行数据的比对工作，例如对账。

下面我们就了解下Java的定时任务调度，会涵盖Timer、ScheduledExecutorService。

## 简介

Java中的定时任务需要Timer、TimerTask来配合完成。

Timer : 一种定时器，线程安排其在后台线程中执行的任务。可以任务执行一次或者定期重复执行。

TimerTask: 定时任务，代表一个可以被Timer执行的任务。

## Timer

定时任务调度：基于给定的时间点、给定的时间间隔、给定的执行次数自动执行的任务。

Timer位于java.util包下，其内部包含且仅包含一个后台线程（TimeThread）对多个业务任务（TimeTask）进行定时定频率的调度。

在工具类Timer中，提供了四个构造方法，每个构造方法都启动了计时器线程，同时Timer类可以保证多个线程可以共享单个Timer对象而无需进行外部同步，所以Timer类是线程安全的。但是由于每一个Timer对象对应的是单个后台线程，用于顺序执行所有的计时器任务，一般情况下我们的线程任务执行所消耗的时间应该非常短，但是由于特殊情况导致某个定时器任务执行的时间太长，那么他就会“独占”计时器的任务执行线程，其后的所有线程都必须等待它执行完，这就会延迟后续任务的执行，使这些任务堆积在一起，具体情况我们后面分析。

当程序初始化完成Timer后，定时任务就会按照我们设定的时间去执行，Timer提供了schedule方法，该方法有多中重载方式来适应不同的情况，如下：

```
schedule(TimerTask task, Date time)：安排在指定的时间执行指定的任务。

schedule(TimerTask task, Date firstTime, long period) ：安排指定的任务在指定的时间开始进行重复的固定延迟执行。

schedule(TimerTask task, long delay) ：安排在指定延迟后执行指定的任务。

schedule(TimerTask task, long delay, long period) ：安排指定的任务从指定的延迟后开始进行重复的固定延迟执行。

同时也重载了scheduleAtFixedRate方法，scheduleAtFixedRate方法与schedule相同，只不过他们的侧重点不同，区别后面分析。

scheduleAtFixedRate(TimerTask task, Date firstTime, long period)：安排指定的任务在指定的时间开始进行重复的固定速率执行。

scheduleAtFixedRate(TimerTask task, long delay, long period)：安排指定的任务在指定的延迟后开始进行重复的固定速率执行。
```

参数说明

```
task：所要执行的任务，需要extends TimeTask override run()

time/firstTime：首次执行任务的时间

period：周期性执行Task的时间间隔，单位是毫秒

delay：执行task任务前的延时时间，单位是毫秒

很显然，通过上述的描述，我们可以实现：

延迟多久后执行一次任务；指定时间执行一次任务；延迟一段时间，并周期性执行任务；指定时间，并周期性执行任务；
```

## TimerTask

TimerTask类是一个抽象类，由Timer 安排为一次执行或重复执行的任务。它有一个抽象方法run()方法，该方法用于执行相应计时器任务要执行的操作。因此每一个具体的任务类都必须继承TimerTask，然后重写run()方法。

另外它还有两个非抽象的方法：

boolean cancel()：取消此计时器任务。

long scheduledExecutionTime()：返回此任务最近实际执行的安排执行时间。

## 分析

**schedule(TimerTask task, Date time)、schedule(TimerTask task, long delay)**

对于这两个方法而言，如果指定的计划执行时间scheduledExecutionTime<= systemCurrentTime，则task会被立即执行。scheduledExecutionTime不会因为某一个task的过度执行而改变。

**schedule(TimerTask task, Date firstTime, long period)、schedule(TimerTask task, long delay, long period)**

这两个方法与上面两个就有点儿不同的，前面提过Timer的计时器任务会因为前一个任务执行时间较长而延时。在这两个方法中，每一次执行的task的计划时间会随着前一个task的实际时间而发生改变，也就是scheduledExecutionTime(n+1)=realExecutionTime(n)+periodTime。也就是说如果第n个task由于某种情况导致这次的执行时间过程，最后导致systemCurrentTime>= scheduledExecutionTime(n+1)，这是第n+1个task并不会因为到时了而执行，他会等待第n个task执行完之后再执行，那么这样势必会导致n+2个的执行实现scheduledExecutionTime放生改变即scheduledExecutionTime(n+2) = realExecutionTime(n+1)+periodTime。所以这两个方法更加注重保存间隔时间的稳定。

**scheduleAtFixedRate(TimerTask task, Date firstTime, long period)、scheduleAtFixedRate(TimerTask task, long delay, long period)**

在前面也提过scheduleAtFixedRate与schedule方法的侧重点不同，schedule方法侧重保存间隔时间的稳定，而scheduleAtFixedRate方法更加侧重于保持执行频率的稳定。为什么这么说，原因如下。在schedule方法中会因为前一个任务的延迟而导致其后面的定时任务延时，而scheduleAtFixedRate方法则不会，如果第n个task执行时间过长导致systemCurrentTime>= scheduledExecutionTime(n+1)，则不会做任何等待他会立即执行第n+1个task，所以scheduleAtFixedRate方法执行时间的计算方法不同于schedule，而是scheduledExecutionTime(n)=firstExecuteTime +n*periodTime，该计算方法永远保持不变。所以scheduleAtFixedRate更加侧重于保持执行频率的稳定。

**思考1：如果time/firstTime指定的时间，在当前时间之前，会发生什么呢？
**

> 在时间等于或者超过time/firstTime的时候，会执行task！也就是说，如果time/firstTime指定的时间在当前时间之前，就会立即得到执行。

**思考2：schedule和scheduleAtFixedRate有什么区别？
**

> scheduleAtFixedRate：每次执行时间为上一次任务开始起向后推一个period间隔，也就是说下次执行时间相对于上一次任务开始的时间点，因此执行时间不会延后，但是存在任务并发执行的问题。
>
> schedule：每次执行时间为上一次任务结束后推一个period间隔，也就是说下次执行时间相对于上一次任务结束的时间点，因此执行时间会不断延后。

**思考3：如果执行task发生异常，是否会影响其他task的定时调度？
**

> 如果TimeTask抛出RuntimeException，那么Timer会停止所有任务的运行！

## Timer的缺陷

Timer计时器可以定时（指定时间执行任务）、延迟（延迟5秒执行任务）、周期性地执行任务（每隔个1秒执行任务），但是，Timer存在一些缺陷。首先Timer对调度的支持是基于绝对时间的，而不是相对时间，所以它对系统时间的改变非常敏感。其次Timer线程是不会捕获异常的，如果TimerTask抛出的了未检查异常则会导致Timer线程终止，同时Timer也不会重新恢复线程的执行，他会错误的认为整个Timer线程都会取消。同时，已经被安排单尚未执行的TimerTask也不会再执行了，新的任务也不能被调度。故如果TimerTask抛出未检查的异常，Timer将会产生无法预料的行为。

## ScheduledExecutorService替代Timer

ScheduledExecutorService中定义的这四个接口方法和 Timer 中对应的方法几乎一样，只不过 Timer 的 scheduled 方法需要在外部传入一个 TimerTask 的抽象任务。

而我们的 ScheduledExecutorService 封装的更加细致了，随便你传 Runnable 或是 Callable，我会在内部给你做一层封装，封装一个类似 TimerTask 的抽象任务类（ScheduledFutureTask）。

然后传入线程池，启动线程去执行该任务，而我们的 ScheduledFutureTask 重写的 run 方法是这样的：

如果 periodic 为 true 则说明这是一个需要重复执行的任务，否则说明是一个一次性任务。

所以实际执行该任务的时候，需要分类，如果是普通的任务就直接调用 run 方法执行即可，否则在执行结束之后还需要重置下下一次执行时间。

整体来说，ScheduledExecutorService 区别于 Timer 的地方就在于前者依赖了线程池来执行任务，而任务本身会判断是什么类型的任务，需要重复执行的在任务执行结束后会被重新添加到任务队列。

而对于后者来说，它只依赖一个线程不停的去获取队列首部的任务并尝试执行它，无论是效率上、还是安全性上都比不上前者。

所以，建议使用 ScheduledExecutorService 取代 Timer，当然，通过学习 Timer 会更有助于对 ScheduledExecutorService 的研究。

## 源码

**Timer**

```
// 定时器
public class Timer {
    /**
     * This ID is used to generate thread names.
     */
    private static final AtomicInteger nextSerialNumber = new AtomicInteger(0);
    
    /**
     * The timer task queue.  This data structure is shared with the timer
     * thread.  The timer produces tasks, via its various schedule calls,
     * and the timer thread consumes, executing timer tasks as appropriate,
     * and removing them from the queue when they're obsolete.
     */
    // 定时器任务队列
    private final TaskQueue queue = new TaskQueue();
    
    /**
     * The timer thread.
     */
    // 定时器线程
    private final TimerThread thread = new TimerThread(queue);
    
    /**
     * This object causes the timer's task execution thread to exit gracefully
     * when there are no live references to the Timer object and no tasks in the timer queue.
     * It is used in preference to a finalizer on Timer as such a finalizer would be susceptible
     * to a subclass's finalizer forgetting to call it.
     */
    private final Object threadReaper = new Object() {
        @SuppressWarnings("deprecation")
        protected void finalize() throws Throwable {
            synchronized(queue) {
                // 取消定时器
                thread.newTasksMayBeScheduled = false;
                // 唤醒定时器线程
                queue.notify(); // In case queue is empty.
            }
        }
    };
    
    
    
    /*▼ 构造方法 ████████████████████████████████████████████████████████████████████████████████┓ */
    
    /**
     * Creates a new timer.  The associated thread does <i>not</i>
     * {@linkplain Thread#setDaemon run as a daemon}.
     */
    public Timer() {
        this("Timer-" + serialNumber());
    }
    
    /**
     * Creates a new timer whose associated thread has the specified name.
     * The associated thread does <i>not</i>
     * {@linkplain Thread#setDaemon run as a daemon}.
     *
     * @param name the name of the associated thread
     *
     * @throws NullPointerException if {@code name} is null
     * @since 1.5
     */
    public Timer(String name) {
        thread.setName(name);
        thread.start();
    }
    
    /**
     * Creates a new timer whose associated thread may be specified to
     * {@linkplain Thread#setDaemon run as a daemon}.
     * A daemon thread is called for if the timer will be used to
     * schedule repeating "maintenance activities", which must be
     * performed as long as the application is running, but should not
     * prolong the lifetime of the application.
     *
     * @param isDaemon true if the associated thread should run as a daemon.
     */
    public Timer(boolean isDaemon) {
        this("Timer-" + serialNumber(), isDaemon);
    }
    
    /**
     * Creates a new timer whose associated thread has the specified name,
     * and may be specified to
     * {@linkplain Thread#setDaemon run as a daemon}.
     *
     * @param name     the name of the associated thread
     * @param isDaemon true if the associated thread should run as a daemon
     *
     * @throws NullPointerException if {@code name} is null
     * @since 1.5
     */
    public Timer(String name, boolean isDaemon) {
        thread.setName(name);
        thread.setDaemon(isDaemon);
        thread.start();
    }
    
    /*▲ 构造方法 ████████████████████████████████████████████████████████████████████████████████┛ */
    
    
    /*▼ 执行任务 ████████████████████████████████████████████████████████████████████████████████┓ */
    
    /**
     * Schedules the specified task for execution after the specified delay.
     *
     * @param task  task to be scheduled.
     * @param delay delay in milliseconds before task is to be executed.
     *
     * @throws IllegalArgumentException if {@code delay} is negative, or
     *                                  {@code delay + System.currentTimeMillis()} is negative.
     * @throws IllegalStateException    if task was already scheduled or
     *                                  cancelled, timer was cancelled, or timer thread terminated.
     * @throws NullPointerException     if {@code task} is null
     */
    // 安排一次性任务在delay延时后开始执行
    public void schedule(TimerTask task, long delay) {
        if(delay<0) {
            throw new IllegalArgumentException("Negative delay.");
        }
        
        sched(task, System.currentTimeMillis() + delay, 0);
    }
    
    /**
     * Schedules the specified task for execution at the specified time.  If
     * the time is in the past, the task is scheduled for immediate execution.
     *
     * @param task task to be scheduled.
     * @param time time at which task is to be executed.
     *
     * @throws IllegalArgumentException if {@code time.getTime()} is negative.
     * @throws IllegalStateException    if task was already scheduled or
     *                                  cancelled, timer was cancelled, or timer thread terminated.
     * @throws NullPointerException     if {@code task} or {@code time} is null
     */
    // 安排一次性任务在time时间时开始执行
    public void schedule(TimerTask task, Date time) {
        sched(task, time.getTime(), 0);
    }
    
    /**
     * Schedules the specified task for repeated <i>fixed-delay execution</i>,
     * beginning after the specified delay.  Subsequent executions take place
     * at approximately regular intervals separated by the specified period.
     *
     * <p>In fixed-delay execution, each execution is scheduled relative to
     * the actual execution time of the previous execution.  If an execution
     * is delayed for any reason (such as garbage collection or other
     * background activity), subsequent executions will be delayed as well.
     * In the long run, the frequency of execution will generally be slightly
     * lower than the reciprocal of the specified period (assuming the system
     * clock underlying {@code Object.wait(long)} is accurate).
     *
     * <p>Fixed-delay execution is appropriate for recurring activities
     * that require "smoothness."  In other words, it is appropriate for
     * activities where it is more important to keep the frequency accurate
     * in the short run than in the long run.  This includes most animation
     * tasks, such as blinking a cursor at regular intervals.  It also includes
     * tasks wherein regular activity is performed in response to human
     * input, such as automatically repeating a character as long as a key
     * is held down.
     *
     * @param task   task to be scheduled.
     * @param delay  delay in milliseconds before task is to be executed.
     * @param period time in milliseconds between successive task executions.
     *
     * @throws IllegalArgumentException if {@code delay < 0}, or
     *                                  {@code delay + System.currentTimeMillis() < 0}, or
     *                                  {@code period <= 0}
     * @throws IllegalStateException    if task was already scheduled or
     *                                  cancelled, timer was cancelled, or timer thread terminated.
     * @throws NullPointerException     if {@code task} is null
     */
    // 安排【固定延时】任务在delay延时后开始执行
    public void schedule(TimerTask task, long delay, long period) {
        if(delay<0) {
            throw new IllegalArgumentException("Negative delay.");
        }
        
        if(period<=0) {
            throw new IllegalArgumentException("Non-positive period.");
        }
        
        sched(task, System.currentTimeMillis() + delay, -period);
    }
    
    /**
     * Schedules the specified task for repeated <i>fixed-delay execution</i>,
     * beginning at the specified time. Subsequent executions take place at
     * approximately regular intervals, separated by the specified period.
     *
     * <p>In fixed-delay execution, each execution is scheduled relative to
     * the actual execution time of the previous execution.  If an execution
     * is delayed for any reason (such as garbage collection or other
     * background activity), subsequent executions will be delayed as well.
     * In the long run, the frequency of execution will generally be slightly
     * lower than the reciprocal of the specified period (assuming the system
     * clock underlying {@code Object.wait(long)} is accurate).  As a
     * consequence of the above, if the scheduled first time is in the past,
     * it is scheduled for immediate execution.
     *
     * <p>Fixed-delay execution is appropriate for recurring activities
     * that require "smoothness."  In other words, it is appropriate for
     * activities where it is more important to keep the frequency accurate
     * in the short run than in the long run.  This includes most animation
     * tasks, such as blinking a cursor at regular intervals.  It also includes
     * tasks wherein regular activity is performed in response to human
     * input, such as automatically repeating a character as long as a key
     * is held down.
     *
     * @param task      task to be scheduled.
     * @param firstTime First time at which task is to be executed.
     * @param period    time in milliseconds between successive task executions.
     *
     * @throws IllegalArgumentException if {@code firstTime.getTime() < 0}, or
     *                                  {@code period <= 0}
     * @throws IllegalStateException    if task was already scheduled or
     *                                  cancelled, timer was cancelled, or timer thread terminated.
     * @throws NullPointerException     if {@code task} or {@code firstTime} is null
     */
    // 安排【固定延时】任务在firstTime时间时开始执行
    public void schedule(TimerTask task, Date firstTime, long period) {
        if(period<=0) {
            throw new IllegalArgumentException("Non-positive period.");
        }
        
        sched(task, firstTime.getTime(), -period);
    }
    
    /**
     * Schedules the specified task for repeated <i>fixed-rate execution</i>,
     * beginning after the specified delay.  Subsequent executions take place
     * at approximately regular intervals, separated by the specified period.
     *
     * <p>In fixed-rate execution, each execution is scheduled relative to the
     * scheduled execution time of the initial execution.  If an execution is
     * delayed for any reason (such as garbage collection or other background
     * activity), two or more executions will occur in rapid succession to
     * "catch up."  In the long run, the frequency of execution will be
     * exactly the reciprocal of the specified period (assuming the system
     * clock underlying {@code Object.wait(long)} is accurate).
     *
     * <p>Fixed-rate execution is appropriate for recurring activities that
     * are sensitive to <i>absolute</i> time, such as ringing a chime every
     * hour on the hour, or running scheduled maintenance every day at a
     * particular time.  It is also appropriate for recurring activities
     * where the total time to perform a fixed number of executions is
     * important, such as a countdown timer that ticks once every second for
     * ten seconds.  Finally, fixed-rate execution is appropriate for
     * scheduling multiple repeating timer tasks that must remain synchronized
     * with respect to one another.
     *
     * @param task   task to be scheduled.
     * @param delay  delay in milliseconds before task is to be executed.
     * @param period time in milliseconds between successive task executions.
     *
     * @throws IllegalArgumentException if {@code delay < 0}, or
     *                                  {@code delay + System.currentTimeMillis() < 0}, or
     *                                  {@code period <= 0}
     * @throws IllegalStateException    if task was already scheduled or
     *                                  cancelled, timer was cancelled, or timer thread terminated.
     * @throws NullPointerException     if {@code task} is null
     */
    // 安排【固定周期】任务在delay延时后开始执行
    public void scheduleAtFixedRate(TimerTask task, long delay, long period) {
        if(delay<0) {
            throw new IllegalArgumentException("Negative delay.");
        }
        
        if(period<=0) {
            throw new IllegalArgumentException("Non-positive period.");
        }
        
        sched(task, System.currentTimeMillis() + delay, period);
    }
    
    /**
     * Schedules the specified task for repeated <i>fixed-rate execution</i>,
     * beginning at the specified time. Subsequent executions take place at
     * approximately regular intervals, separated by the specified period.
     *
     * <p>In fixed-rate execution, each execution is scheduled relative to the
     * scheduled execution time of the initial execution.  If an execution is
     * delayed for any reason (such as garbage collection or other background
     * activity), two or more executions will occur in rapid succession to
     * "catch up."  In the long run, the frequency of execution will be
     * exactly the reciprocal of the specified period (assuming the system
     * clock underlying {@code Object.wait(long)} is accurate).  As a
     * consequence of the above, if the scheduled first time is in the past,
     * then any "missed" executions will be scheduled for immediate "catch up"
     * execution.
     *
     * <p>Fixed-rate execution is appropriate for recurring activities that
     * are sensitive to <i>absolute</i> time, such as ringing a chime every
     * hour on the hour, or running scheduled maintenance every day at a
     * particular time.  It is also appropriate for recurring activities
     * where the total time to perform a fixed number of executions is
     * important, such as a countdown timer that ticks once every second for
     * ten seconds.  Finally, fixed-rate execution is appropriate for
     * scheduling multiple repeating timer tasks that must remain synchronized
     * with respect to one another.
     *
     * @param task      task to be scheduled.
     * @param firstTime First time at which task is to be executed.
     * @param period    time in milliseconds between successive task executions.
     *
     * @throws IllegalArgumentException if {@code firstTime.getTime() < 0} or {@code period <= 0}
     * @throws IllegalStateException    if task was already scheduled or cancelled,
     *                                  timer was cancelled, or timer thread terminated.
     * @throws NullPointerException     if {@code task} or {@code firstTime} is null
     */
    // 安排【固定周期】任务在firstTime时间时开始执行
    public void scheduleAtFixedRate(TimerTask task, Date firstTime, long period) {
        if(period<=0) {
            throw new IllegalArgumentException("Non-positive period.");
        }
        
        sched(task, firstTime.getTime(), period);
    }
    
    /**
     * Schedule the specified timer task for execution at the specified
     * time with the specified period, in milliseconds.  If period is
     * positive, the task is scheduled for repeated execution; if period is
     * zero, the task is scheduled for one-time execution. Time is specified
     * in Date.getTime() format.  This method checks timer state, task state,
     * and initial execution time, but not period.
     *
     * @throws IllegalArgumentException if {@code time} is negative.
     * @throws IllegalStateException    if task was already scheduled or
     *                                  cancelled, timer was cancelled, or timer thread terminated.
     * @throws NullPointerException     if {@code task} is null
     */
    /*
     * 安排任务以待执行
     *
     * time  ：任务触发时间
     *
     * period：任务的重复模式：
     *         零：非重复任务：只执行一次
     *         正数：重复性任务：固定周期，从任务初次被触发开始，以后每隔period时间就被触发一次
     *         负数：重复性任务：固定延时，任务下次的开始时间=任务上次结束时间+(-period)
     */
    private void sched(TimerTask task, long time, long period) {
        if(time<0) {
            throw new IllegalArgumentException("Illegal execution time.");
        }
        
        // Constrain value of period sufficiently to prevent numeric
        // overflow while still being effectively infinitely large.
        if(Math.abs(period)>(Long.MAX_VALUE >> 1)) {
            period >>= 1;
        }
        
        synchronized(queue) {
            if(!thread.newTasksMayBeScheduled) {
                throw new IllegalStateException("Timer already cancelled.");
            }
            
            synchronized(task.lock) {
                // 如果任务已经开始执行或已被取消，抛出异常
                if(task.state != TimerTask.VIRGIN) {
                    throw new IllegalStateException("Task already scheduled or cancelled");
                }
                
                // 记录任务触发时间
                task.nextExecutionTime = time;
                
                // 任务的重复模式
                task.period = period;
                
                // 任务进入【排队】状态
                task.state = TimerTask.SCHEDULED;
            }
            
            // 将任务加入任务队列
            queue.add(task);
    
            // 如果当前任务排在队头
            if(queue.getMin() == task) {
                // 唤醒定时器线程
                queue.notify();
            }
        }
    }
    /*▲ 执行任务 ████████████████████████████████████████████████████████████████████████████████┛ */
    
    
    
    /*▼ 取消定时器 ████████████████████████████████████████████████████████████████████████████████┓ */
    
    /**
     * Terminates this timer, discarding any currently scheduled tasks.
     * Does not interfere with a currently executing task (if it exists).
     * Once a timer has been terminated, its execution thread terminates
     * gracefully, and no more tasks may be scheduled on it.
     *
     * <p>Note that calling this method from within the run method of a
     * timer task that was invoked by this timer absolutely guarantees that
     * the ongoing task execution is the last task execution that will ever
     * be performed by this timer.
     *
     * <p>This method may be called repeatedly; the second and subsequent
     * calls have no effect.
     */
    // 取消定时器
    public void cancel() {
        synchronized(queue) {
            // 不再执行任务
            thread.newTasksMayBeScheduled = false;
            // 清空任务队列
            queue.clear();
            // 唤醒定时器线程
            queue.notify();  // In case queue was already empty.
        }
    }
    
    /**
     * Removes all cancelled tasks from this timer's task queue.  <i>Calling
     * this method has no effect on the behavior of the timer</i>, but
     * eliminates the references to the cancelled tasks from the queue.
     * If there are no external references to these tasks, they become
     * eligible for garbage collection.
     *
     * <p>Most programs will have no need to call this method.
     * It is designed for use by the rare application that cancels a large
     * number of tasks.  Calling this method trades time for space: the
     * runtime of the method may be proportional to n + c log n, where n
     * is the number of tasks in the queue and c is the number of cancelled
     * tasks.
     *
     * <p>Note that it is permissible to call this method from within
     * a task scheduled on this timer.
     *
     * @return the number of tasks removed from the queue.
     *
     * @since 1.5
     */
    // 清除所有进入【取消】状态的任务
    public int purge() {
        int result = 0;
        
        synchronized(queue) {
            // 遍历所有任务
            for(int i = queue.size(); i>0; i--) {
                // 找出已取消的任务
                if(queue.get(i).state == TimerTask.CANCELLED) {
                    // 快速移除索引i处的任务（没有重建小顶堆）
                    queue.quickRemove(i);
                    result++;
                }
            }
            
            // 将重建小顶堆的时间推迟到这里
            if(result != 0) {
                // 重建小顶堆
                queue.heapify();
            }
        }
        
        return result;
    }
    
    /*▲ 取消定时器  */
    
   
   
    // 返回一个定时器编号
    private static int serialNumber() {
        return nextSerialNumber.getAndIncrement();
    }
    
}
```

**TaskQueue**

```
package java.util;

/**
 * This class represents a timer task queue: a priority queue of TimerTasks,
 * ordered on nextExecutionTime.  Each Timer object has one of these, which it
 * shares with its TimerThread.  Internally this class uses a heap, which
 * offers log(n) performance for the add, removeMin and rescheduleMin
 * operations, and constant time performance for the getMin operation.
 */
// 定时器任务队列
class TaskQueue {
    /**
     * Priority queue represented as a balanced binary heap: the two children
     * of queue[n] are queue[2*n] and queue[2*n+1].  The priority queue is
     * ordered on the nextExecutionTime field: The TimerTask with the lowest
     * nextExecutionTime is in queue[1] (assuming the queue is nonempty).  For
     * each node n in the heap, and each descendant of n, d,
     * n.nextExecutionTime <= d.nextExecutionTime.
     */
    // 任务队列，实际存储任务的地方，索引0处空闲
    private TimerTask[] queue = new TimerTask[128];
    
    /**
     * The number of tasks in the priority queue.  (The tasks are stored in
     * queue[1] up to queue[size]).
     */
    // 任务数量
    private int size = 0;
    
    /**
     * Adds a new task to the priority queue.
     */
    // 将任务送入任务队列排队
    void add(TimerTask task) {
        // Grow backing store if necessary
        if(size + 1 == queue.length) {
            // 扩容
            queue = Arrays.copyOf(queue, 2 * queue.length);
        }
        
        queue[++size] = task;
    
        // 调整size处的任务到队列中的合适位置
        fixUp(size);
    }
    
    /**
     * Return the "head task" of the priority queue.
     * (The head task is an task with the lowest nextExecutionTime.)
     */
    // 获取队头任务
    TimerTask getMin() {
        return queue[1];
    }
    
    /**
     * Return the ith task in the priority queue, where i ranges from 1 (the
     * head task, which is returned by getMin) to the number of tasks on the
     * queue, inclusive.
     */
    // 获取索引i处的任务
    TimerTask get(int i) {
        return queue[i];
    }
    
    /**
     * Remove the head task from the priority queue.
     */
    // 移除队头任务，并将触发时间最近的任务放在队头
    void removeMin() {
        // 先将队尾任务放到队头
        queue[1] = queue[size];
        queue[size--] = null;  // Drop extra reference to prevent memory leak
        // 调整当前队头任务（之前的队尾任务）到队列中合适的位置
        fixDown(1);
    }
    
    /**
     * Removes the ith element from queue without regard for maintaining
     * the heap invariant.  Recall that queue is one-based, so
     * 1 <= i <= size.
     */
    // 快速移除索引i处的任务（没有重建小顶堆）
    void quickRemove(int i) {
        assert i<=size;
        
        queue[i] = queue[size];
        queue[size--] = null;  // Drop extra ref to prevent memory leak
    }
    
    /**
     * Sets the nextExecutionTime associated with the head task to the
     * specified value, and adjusts priority queue accordingly.
     */
    // 重置队头任务的触发时间，并将其调整到队列中的合适位置
    void rescheduleMin(long newTime) {
        // 重置队头任务的触发时间
        queue[1].nextExecutionTime = newTime;
        // 将该任务调整到队列中的合适位置
        fixDown(1);
    }
    
    /**
     * Removes all elements from the priority queue.
     */
    // 清空任务队列
    void clear() {
        // Null out task references to prevent memory leak
        for(int i = 1; i<=size; i++) {
            queue[i] = null;
        }
        
        size = 0;
    }
    
    /**
     * Returns true if the priority queue contains no elements.
     */
    // 判断队列是否为空
    boolean isEmpty() {
        return size == 0;
    }
    
    /**
     * Returns the number of tasks currently on the queue.
     */
    // 返回队列长度
    int size() {
        return size;
    }
    
    /**
     * Establishes the heap invariant (described above) in the entire tree,
     * assuming nothing about the order of the elements prior to the call.
     */
    // 重建小顶堆
    void heapify() {
        for(int i = size / 2; i >= 1; i--) {
            fixDown(i);
        }
    }
    
    /**
     * Establishes the heap invariant (described above) assuming the heap
     * satisfies the invariant except possibly for the leaf-node indexed by k
     * (which may have a nextExecutionTime less than its parent's).
     *
     * This method functions by "promoting" queue[k] up the hierarchy
     * (by swapping it with its parent) repeatedly until queue[k]'s
     * nextExecutionTime is greater than or equal to that of its parent.
     */
    // 插入。需要从小顶堆的结点k开始，向【上】查找一个合适的位置插入原k索引处的任务
    private void fixUp(int k) {
        while(k>1) {
            // 获取父结点索引
            int j = k >> 1;
    
            // 如果待插入元素大于父节点中的元素，则退出循环
            if(queue[k].nextExecutionTime>=queue[j].nextExecutionTime) {
                break;
            }
    
            // 子结点保存父结点中的元素
            TimerTask tmp = queue[j];
            queue[j] = queue[k];
            queue[k] = tmp;
    
            // 向上搜寻合适的插入位置
            k = j;
        }
    }
    
    /**
     * Establishes the heap invariant (described above) in the subtree
     * rooted at k, which is assumed to satisfy the heap invariant except
     * possibly for node k itself (which may have a nextExecutionTime greater
     * than its children's).
     *
     * This method functions by "demoting" queue[k] down the hierarchy
     * (by swapping it with its smaller child) repeatedly until queue[k]'s
     * nextExecutionTime is less than or equal to those of its children.
     */
    // 插入。需要从小顶堆的结点k开始，向【下】查找一个合适的位置插入原k索引处的任务
    private void fixDown(int k) {
        int j;
        
        while((j = k << 1)<=size && j>0) {
            // 让j存储子结点中较小结点的索引
            if(j<size && queue[j].nextExecutionTime>queue[j + 1].nextExecutionTime) {
                j++; // j indexes smallest kid
            }
    
            // 如果待插入元素小于子结点中较小的元素，则退出循环
            if(queue[k].nextExecutionTime<=queue[j].nextExecutionTime) {
                break;
            }
    
            // 父结点位置保存子结点中较小的元素
            TimerTask tmp = queue[j];
            queue[j] = queue[k];
            queue[k] = tmp;
    
            // 向下搜寻合适的插入位置
            k = j;
        }
    }
}
```

**TimerTask**

```
/*
 * Copyright (c) 1999, 2013, Oracle and/or its affiliates. All rights reserved.
 * DO NOT ALTER OR REMOVE COPYRIGHT NOTICES OR THIS FILE HEADER.
 *
 * This code is free software; you can redistribute it and/or modify it
 * under the terms of the GNU General Public License version 2 only, as
 * published by the Free Software Foundation.  Oracle designates this
 * particular file as subject to the "Classpath" exception as provided
 * by Oracle in the LICENSE file that accompanied this code.
 *
 * This code is distributed in the hope that it will be useful, but WITHOUT
 * ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or
 * FITNESS FOR A PARTICULAR PURPOSE.  See the GNU General Public License
 * version 2 for more details (a copy is included in the LICENSE file that
 * accompanied this code).
 *
 * You should have received a copy of the GNU General Public License version
 * 2 along with this work; if not, write to the Free Software Foundation,
 * Inc., 51 Franklin St, Fifth Floor, Boston, MA 02110-1301 USA.
 *
 * Please contact Oracle, 500 Oracle Parkway, Redwood Shores, CA 94065 USA
 * or visit www.oracle.com if you need additional information or have any
 * questions.
 */

package java.util;

/**
 * A task that can be scheduled for one-time or repeated execution by a
 * {@link Timer}.
 *
 * <p>A timer task is <em>not</em> reusable.  Once a task has been scheduled
 * for execution on a {@code Timer} or cancelled, subsequent attempts to
 * schedule it for execution will throw {@code IllegalStateException}.
 *
 * @author Josh Bloch
 * @since 1.3
 */
// 定时任务
public abstract class TimerTask implements Runnable {
    /**
     * This task has not yet been scheduled.
     */
    static final int VIRGIN    = 0; // 【初始化】
    /**
     * This task is scheduled for execution.  If it is a non-repeating task,
     * it has not yet been executed.
     */
    static final int SCHEDULED = 1; // 【排队】
    /**
     * This non-repeating task has already executed (or is currently
     * executing) and has not been cancelled.
     */
    static final int EXECUTED  = 2; // 【执行】
    /**
     * This task has been cancelled (with a call to TimerTask.cancel).
     */
    static final int CANCELLED = 3; // 【取消】
    
    /**
     * The state of this task, chosen from the constants below.
     */
    int state = VIRGIN; // 任务状态
    
    /**
     * Next execution time for this task in the format returned by
     * System.currentTimeMillis, assuming this task is scheduled for execution.
     * For repeating tasks, this field is updated prior to each task execution.
     */
    // 任务触发时间
    long nextExecutionTime;
    
    /**
     * Period in milliseconds for repeating tasks.  A positive value indicates
     * fixed-rate execution.  A negative value indicates fixed-delay execution.
     * A value of 0 indicates a non-repeating task.
     */
    /*
     * 任务的重复模式：
     *   零：非重复任务：只执行一次
     * 正数：重复性任务：固定周期，从任务初次被触发开始，以后每隔period时间就被触发一次
     * 负数：重复性任务：固定延时，任务下次的开始时间=任务上次结束时间+(-period)
     */
    long period = 0;
    
    /**
     * This object is used to control access to the TimerTask internals.
     */
    final Object lock = new Object();
    
    /**
     * Creates a new timer task.
     */
    protected TimerTask() {
    }
    
    /**
     * The action to be performed by this timer task.
     */
    // 执行任务
    public abstract void run();
    
    /**
     * Cancels this timer task.  If the task has been scheduled for one-time
     * execution and has not yet run, or has not yet been scheduled, it will
     * never run.  If the task has been scheduled for repeated execution, it
     * will never run again.  (If the task is running when this call occurs,
     * the task will run to completion, but will never run again.)
     *
     * <p>Note that calling this method from within the {@code run} method of
     * a repeating timer task absolutely guarantees that the timer task will
     * not run again.
     *
     * <p>This method may be called repeatedly; the second and subsequent
     * calls have no effect.
     *
     * @return true if this task is scheduled for one-time execution and has
     * not yet run, or this task is scheduled for repeated execution.
     * Returns false if the task was scheduled for one-time execution
     * and has already run, or if the task was never scheduled, or if
     * the task was already cancelled.  (Loosely speaking, this method
     * returns {@code true} if it prevents one or more scheduled
     * executions from taking place.)
     */
    // 取消处于【排队】状态的任务
    public boolean cancel() {
        synchronized(lock) {
            // 如果任务处于【排队】状态，则可以取消
            boolean result = (state == SCHEDULED);
            // 任务进入【取消】状态
            state = CANCELLED;
            return result;
        }
    }
    
    /**
     * Returns the <i>scheduled</i> execution time of the most recent
     * <i>actual</i> execution of this task.  (If this method is invoked
     * while task execution is in progress, the return value is the scheduled
     * execution time of the ongoing task execution.)
     *
     * <p>This method is typically invoked from within a task's run method, to
     * determine whether the current execution of the task is sufficiently
     * timely to warrant performing the scheduled activity:
     * <pre>{@code
     *   public void run() {
     *       if (System.currentTimeMillis() - scheduledExecutionTime() >=
     *           MAX_TARDINESS)
     *               return;  // Too late; skip this execution.
     *       // Perform the task
     *   }
     * }</pre>
     * This method is typically <i>not</i> used in conjunction with
     * <i>fixed-delay execution</i> repeating tasks, as their scheduled
     * execution times are allowed to drift over time, and so are not terribly
     * significant.
     *
     * @return the time at which the most recent execution of this task was
     * scheduled to occur, in the format returned by Date.getTime().
     * The return value is undefined if the task has yet to commence
     * its first execution.
     *
     * @see Date#getTime()
     */
    // 返回任务被安排去【排队】时的时间
    public long scheduledExecutionTime() {
        synchronized(lock) {
            return (period<0
                ? nextExecutionTime + period
                : nextExecutionTime - period);
        }
    }
}
```

