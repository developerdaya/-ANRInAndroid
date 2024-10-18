# -ANRInAndroid
In Android, an **ANR (Application Not Responding)** error occurs when the main thread (UI thread) of an application is blocked for too long (usually more than **5 seconds**). ANR events can happen due to various reasons, often related to poor app design or long-running operations on the main thread.

Here are the main types of ANR that can occur in Android:

### 1. **KeyDispatch Timeout (5 seconds)**
   This happens when the **UI thread is blocked** and the system cannot process user input events (e.g., touch or key events) within 5 seconds. 
   
   **Common Causes:**
   - Performing long-running operations (e.g., network requests, file I/O, database operations) directly on the **UI thread**.
   - Synchronization issues (e.g., waiting for a lock or a thread to finish on the UI thread).
   - Heavy operations like loading large images or processing large datasets in the main thread.
   
   **How to Prevent:**
   - Move time-consuming tasks (e.g., network, database, or file operations) to background threads using mechanisms like **AsyncTask**, **Thread**, **Executor**, or **Coroutines**.
   - Use the **Handler** or **post()** method to schedule UI updates after background tasks are done.

### 2. **BroadcastReceiver Timeout (10 seconds)**
   If a **BroadcastReceiver** does not finish executing its `onReceive()` method within **10 seconds**, it triggers an ANR.

   **Common Causes:**
   - Performing long tasks such as database operations or network requests inside the `onReceive()` method.
   - Trying to update the UI directly from the `onReceive()` method.

   **How to Prevent:**
   - Keep the logic inside `onReceive()` as minimal as possible. Offload long-running operations to a background service (e.g., **IntentService**, **JobIntentService**).
   - Use a **WorkManager** or **JobScheduler** to schedule background work.

### 3. **Service Timeout**
   ANR can occur when a **Service** (foreground or background) takes too long to execute. The timeout thresholds for services are:
   - **Foreground Service Timeout**: **20 seconds**
   - **Background Service Timeout**: **200 seconds (3 minutes and 20 seconds)**

   **Common Causes:**
   - Running long tasks on the **main thread** inside a Service, blocking the system from processing other requests.
   - Performing synchronous network operations or file I/O on the main thread inside the Service.

   **How to Prevent:**
   - Move heavy operations to a background thread using mechanisms like **IntentService**, **AsyncTask**, **Threads**, or **Coroutines**.
   - Use **JobScheduler** or **WorkManager** for background work.

### 4. **ContentProvider Timeout (10 seconds)**
   When a **ContentProvider** takes too long (more than 10 seconds) to handle a request from another process, it triggers an ANR.

   **Common Causes:**
   - Performing long database queries, network requests, or file I/O operations in the `ContentProvider`'s `query()`, `insert()`, `delete()`, or `update()` methods.

   **How to Prevent:**
   - Use asynchronous processing (e.g., **CursorLoader**, **AsyncTaskLoader**) to handle data loading from the `ContentProvider`.
   - Offload long-running database operations to a background thread.

### 5. **InputDispatching Timeout (5 seconds)**
   This happens when the system is unable to dispatch input events (e.g., touch events, key presses) to the app because the **main thread is blocked**.

   **Common Causes:**
   - Long computations or blocking I/O operations (like reading files, networking, or database access) on the main thread.
   - Synchronous operations that do not return control to the main thread in a timely manner.

   **How to Prevent:**
   - Avoid performing long-running tasks on the UI thread.
   - Use **Coroutines** or **RxJava** to move long-running tasks to background threads.
   - Ensure all input events are processed quickly (ideally in under 16 milliseconds).

### 6. **Window Leaked Timeout**
   This occurs when an app tries to update or modify UI elements (like showing or dismissing dialogs) after the `Activity` has been destroyed.

   **Common Causes:**
   - Trying to display a **Dialog** or update the UI after an `Activity` has been finished.
   - Long-running operations holding references to destroyed activities or fragments, leading to memory leaks.

   **How to Prevent:**
   - Ensure that dialogs or UI operations are dismissed before the activity or fragment is destroyed.
   - Use **WeakReferences** or **LifecycleObservers** to safely handle UI operations tied to the lifecycle of the activity or fragment.

### 7. **Database Operation Timeout**
   ANR can occur when a database operation (e.g., `SQLiteDatabase`, `Room`) runs for too long on the **main thread**. Android discourages performing heavy database queries on the main thread.

   **Common Causes:**
   - Large, complex database queries (e.g., querying thousands of rows) on the UI thread.
   - Writing large amounts of data synchronously on the main thread.

   **How to Prevent:**
   - Always perform database operations in the background using **Room's LiveData**, **Coroutines**, or **AsyncTask**.
   - Use **Room's Paging Library** for efficient data loading and display in a paginated way.

### 8. **Deadlock**
   If two or more threads are waiting for each other to release a resource (e.g., a synchronized block or lock), a **deadlock** can occur, blocking the main thread and causing an ANR.

   **Common Causes:**
   - Incorrect use of **synchronized** blocks, causing threads to wait indefinitely.
   - Using blocking locks like `Thread.join()` or `Object.wait()` inappropriately.

   **How to Prevent:**
   - Avoid deadlocks by careful synchronization management.
   - Use **ReentrantLock** or **tryLock()** for non-blocking lock management.
   - Use **Coroutines** for efficient, non-blocking concurrent tasks.

### 9. **Rendering Too Many Views**
   If the app has a very complex layout with a large number of nested views, it may take too long to render, causing the main thread to block and resulting in an ANR.

   **Common Causes:**
   - Deeply nested layouts (e.g., multiple `LinearLayout`, `RelativeLayout` combinations).
   - Inflating too many views at once, especially in `RecyclerView` or `ListView`.

   **How to Prevent:**
   - Use efficient layout structures (e.g., **ConstraintLayout** or **RecyclerView**).
   - Use **ViewStubs** for lazily loading complex views.
   - Optimize your UI rendering with **Layout Inspector** and **Profile GPU Rendering** tools.

### 10. **Network Operation on Main Thread**
   Performing synchronous network operations (HTTP requests, WebSocket connections, etc.) on the main thread can block it and cause an ANR.

   **Common Causes:**
   - Directly performing network requests (e.g., using `HttpUrlConnection`, `Retrofit` without `async` methods) on the main thread.
   - Using synchronous networking libraries without moving operations to background threads.

   **How to Prevent:**
   - Move network operations to a background thread using **Coroutines**, **AsyncTask**, or **WorkManager**.
   - Always use asynchronous methods for networking (e.g., Retrofit’s `enqueue()` method).

---

### **General Tips to Avoid ANRs:**
- **Avoid Long Operations on Main Thread**: Always move time-consuming tasks off the main thread (use background threads).
- **Use Background Threading Libraries**: Use **Coroutines**, **RxJava**, **AsyncTask**, **HandlerThread**, or **Executors** for non-blocking tasks.
- **Optimize UI Rendering**: Simplify layouts and avoid excessive nested views.
- **Profile the App**: Use **Android Profiler**, **StrictMode**, and **Systrace** to identify performance bottlenecks.
- **Handle UI Responsiveness**: Break down long tasks into smaller parts and update the UI progressively.

Here's a list of **100 ANRs** with short descriptions and how to prevent each:

1. **Long DB query on UI thread**  
   **Prevent**: Use `Room` with coroutines or background threads.

2. **Network request on main thread**  
   **Prevent**: Move network calls to background using `Retrofit.enqueue()` or `Coroutines`.

3. **Too many nested views in layout**  
   **Prevent**: Use `ConstraintLayout` for simpler layouts.

4. **Heavy JSON parsing on UI thread**  
   **Prevent**: Parse JSON using `Gson` or `Moshi` on background threads.

5. **Large bitmap loading on UI thread**  
   **Prevent**: Use `Glide` or `Picasso` for image loading.

6. **File I/O on main thread**  
   **Prevent**: Use `AsyncTask`, `Executor`, or coroutines for file operations.

7. **Synchronized block causing deadlock**  
   **Prevent**: Avoid heavy synchronization on the main thread.

8. **UI update in background thread**  
   **Prevent**: Post UI updates on the main thread using `Handler` or `LiveData`.

9. **Slow RecyclerView binding**  
   **Prevent**: Use `ListAdapter` for efficient data binding.

10. **Blocking network access in BroadcastReceiver**  
    **Prevent**: Offload work to `WorkManager` or `IntentService`.

11. **Loading too much data at once in RecyclerView**  
    **Prevent**: Use paging with `Paging Library`.

12. **Large cursor query on UI thread**  
    **Prevent**: Perform queries on background threads with `CursorLoader`.

13. **Slow image decoding in onCreate()**  
    **Prevent**: Load images asynchronously using `Glide`.

14. **Delay due to database migrations**  
    **Prevent**: Execute migrations in the background during app start-up.

15. **Executing HTTP request synchronously**  
    **Prevent**: Use asynchronous HTTP clients like `OkHttp` with coroutines.

16. **Main thread waiting on Thread.join()**  
    **Prevent**: Avoid blocking the UI thread with `Thread.join()`.

17. **Heavy computation on main thread**  
    **Prevent**: Move computations to a background thread or use coroutines.

18. **Blocking main thread with Thread.sleep()**  
    **Prevent**: Avoid `Thread.sleep()` on the UI thread, use handlers or coroutines.

19. **Rendering large views in onDraw()**  
    **Prevent**: Optimize `onDraw()` to reduce drawing operations.

20. **Large database insert on main thread**  
    **Prevent**: Use coroutines or `AsyncTask` for database operations.

21. **Waiting on AsyncTask result on UI thread**  
    **Prevent**: Use coroutines or callbacks to handle results asynchronously.

22. **Using StrictMode violations on main thread**  
    **Prevent**: Handle warnings from `StrictMode` and avoid I/O operations.

23. **Loading too many fragments at once**  
    **Prevent**: Use lazy loading for fragments to load them on demand.

24. **Blocking main thread with long animations**  
    **Prevent**: Use hardware-accelerated animations with `ObjectAnimator`.

25. **Heavy loop computation in Activity lifecycle**  
    **Prevent**: Move loops to background threads using `Executor`.

26. **Heavy XML layout inflation**  
    **Prevent**: Use `ViewStub` for lazy layout inflation.

27. **Resource-intensive BroadcastReceiver execution**  
    **Prevent**: Keep the `onReceive()` logic minimal, move heavy work to services.

28. **Dialog operation after Activity is destroyed**  
    **Prevent**: Ensure dialogs are dismissed in `onDestroy()`.

29. **Deadlock due to multiple locks**  
    **Prevent**: Use `ReentrantLock` with `tryLock()` to avoid blocking.

30. **Blocking UI with synchronous SQLite operations**  
    **Prevent**: Use `Room` database with coroutines for asynchronous queries.

31. **Too many network retries in main thread**  
    **Prevent**: Use exponential backoff for retries in background.

32. **Camera API processing on UI thread**  
    **Prevent**: Use background threads for camera processing.

33. **Heavy Bitmap transformation on UI thread**  
    **Prevent**: Offload bitmap processing to background threads.

34. **Large audio/video file processing in main thread**  
    **Prevent**: Use `MediaPlayer` or `ExoPlayer` for asynchronous media processing.

35. **UI update in Service thread**  
    **Prevent**: Use `Handler` to post updates from services to the main thread.

36. **Blocking UI with synchronous WorkManager tasks**  
    **Prevent**: Execute `WorkManager` tasks asynchronously.

37. **Large database transaction in onCreate()**  
    **Prevent**: Perform transactions asynchronously or after Activity is created.

38. **Excessive layout re-measuring in RecyclerView**  
    **Prevent**: Optimize layout measuring with stable item heights.

39. **Deeply nested LinearLayouts**  
    **Prevent**: Use `ConstraintLayout` to flatten view hierarchy.

40. **Heavy fragment transactions in onResume()**  
    **Prevent**: Perform fragment transactions asynchronously or defer them.

41. **Complex animations slowing down UI**  
    **Prevent**: Optimize or reduce animation complexity using `Lottie`.

42. **Waiting on Future.get() on main thread**  
    **Prevent**: Avoid blocking with `Future.get()`, use `CompletableFuture`.

43. **Loading a large dataset into memory**  
    **Prevent**: Load data in chunks or pages to avoid memory overload.

44. **Long-running Service tasks in main thread**  
    **Prevent**: Use background threads for long Service tasks.

45. **Synchronous Bluetooth operations**  
    **Prevent**: Perform Bluetooth operations asynchronously.

46. **UI thread blocking due to slow network**  
    **Prevent**: Perform network calls asynchronously and show a loading indicator.

47. **Heavy file download on UI thread**  
    **Prevent**: Use `DownloadManager` or perform downloads in background threads.

48. **Executing long JavaScript in WebView on UI thread**  
    **Prevent**: Use `evaluateJavascript()` asynchronously.

49. **Using Handler.postDelayed() for long waits**  
    **Prevent**: Move long waits to background threads with coroutines.

50. **Large SharedPreferences write on main thread**  
    **Prevent**: Write SharedPreferences data asynchronously.

51. **BroadcastReceiver blocking UI with long processing**  
    **Prevent**: Move long tasks to `IntentService` or background threads.

52. **Fragment's heavy UI operations during transaction**  
    **Prevent**: Keep fragment initialization lightweight.

53. **Heavy logging during app startup**  
    **Prevent**: Avoid excessive logging or move logging to a background thread.

54. **Heavy reflection usage on main thread**  
    **Prevent**: Minimize reflection or cache reflection results.

55. **Long delay due to lifecycle methods of multiple Activities**  
    **Prevent**: Simplify Activity lifecycle operations to avoid blocking.

56. **Excessive GC (Garbage Collection) causing delays**  
    **Prevent**: Optimize memory usage to reduce frequent garbage collection.

57. **Rendering large complex views in ListView/RecyclerView**  
    **Prevent**: Use `ViewHolder` pattern and lazy load views in `RecyclerView`.

58. **Complex custom view rendering on UI thread**  
    **Prevent**: Simplify view rendering logic and move complex tasks to background threads.

59. **Loading large resources (e.g., images) in onCreate()**  
    **Prevent**: Load resources in background threads or use placeholders.

60. **Performing time-consuming work in onActivityResult()**  
    **Prevent**: Offload heavy tasks from `onActivityResult()` to background.

61. **Large HTTP responses processed on main thread**  
    **Prevent**: Use background threads or coroutines to handle large responses.

62. **Synchronous WebSocket communication**  
    **Prevent**: Use asynchronous WebSocket libraries like `OkHttp WebSocket`.

63. **BroadcastReceiver performing database operations**  
    **Prevent**: Move database work to background threads or `WorkManager`.

64. **Heavy startup operations in Application class**  
    **Prevent**: Delay non-essential tasks to be performed asynchronously after startup.

65. **Complex RecyclerView animations**  
    **Prevent**: Optimize or disable complex animations in `RecyclerView`.

66. **Performing DNS lookup on main thread**  
    **Prevent**: Use background threads or `AsyncTask` for DNS lookups.

67. **Too many UI updates in quick succession**  
    **Prevent**: Use `Handler` with post delay or `LiveData` with throttling.

68. **Excessive use of View.invalidate()**  
    **Prevent**: Call `invalidate()` only when necessary.

69. **Network timeout causing UI delay**  
    **Prevent**: Set proper timeouts and handle them in background threads.

70. **Custom view taking too long to render**  
    **Prevent**: Optimize rendering in custom views to reduce draw time.

71. **Large task in onSaveInstanceState()**  
    **Prevent**: Keep `onSaveInstanceState()` lightweight.

72. **Loading multiple fragments at once**  
    **Prevent**: Load fragments lazily or defer fragment initialization.

73. **Synchronous GPS location fetching**  
    **Prevent**: Use background threads to fetch GPS data asynchronously.

74. **Excessive binder transactions in services**  
    **Prevent**: Batch binder transactions or move work to background threads.

75. **Loading large web

 pages in WebView**  
    **Prevent**: Optimize web content or load in smaller parts.

76. **Performing heavy computations during Activity transitions**  
    **Prevent**: Move computations out of Activity transitions into background tasks.

77. **Too many SQLite database operations at once**  
    **Prevent**: Batch operations or spread them across multiple transactions.

78. **Long-running alarm logic on UI thread**  
    **Prevent**: Move alarm processing to `JobScheduler` or `WorkManager`.

79. **Blocking main thread with dialog.dismiss()**  
    **Prevent**: Ensure `dismiss()` is called in the right lifecycle states.

80. **Slow file encryption/decryption on main thread**  
    **Prevent**: Perform encryption/decryption in background threads.

81. **Executing Firebase database calls on UI thread**  
    **Prevent**: Use `Firebase` queries asynchronously.

82. **Synchronous content provider queries**  
    **Prevent**: Use background threads for content provider queries.

83. **Heavy object creation during scrolling**  
    **Prevent**: Reuse objects and avoid frequent allocations during scroll.

84. **FragmentManager.commit() causing UI lag**  
    **Prevent**: Use `commitAllowingStateLoss()` or defer fragment transactions.

85. **Loading ads in WebView synchronously**  
    **Prevent**: Load ads asynchronously to avoid blocking UI.

86. **Calling startActivityForResult() in quick succession**  
    **Prevent**: Limit multiple `startActivityForResult()` calls and stagger them.

87. **Long delay in onClickListener()**  
    **Prevent**: Offload heavy work in `onClick()` to background threads.

88. **Expensive calculation in onGlobalLayoutListener()**  
    **Prevent**: Simplify layout calculations and use view post-delayed tasks.

89. **Slow database cursor operation in list adapters**  
    **Prevent**: Use `CursorLoader` or load data in chunks in background.

90. **Blocking the main thread with long touch events**  
    **Prevent**: Process touch events quickly, or use `GestureDetector`.

91. **Heavy reflection usage during runtime**  
    **Prevent**: Minimize reflection calls and cache them.

92. **Multiple threads accessing SharedPreferences simultaneously**  
    **Prevent**: Use `apply()` instead of `commit()` to avoid blocking.

93. **Synchronous job scheduling in onCreate()**  
    **Prevent**: Schedule jobs asynchronously using `JobScheduler` or `WorkManager`.

94. **Too many async tasks running simultaneously**  
    **Prevent**: Limit the number of `AsyncTask` executions and use `Executor`.

95. **Main thread blocking due to slow certificate verification**  
    **Prevent**: Handle SSL certificate verification asynchronously.

96. **Complex view tree in RecyclerView**  
    **Prevent**: Simplify the view hierarchy or use view holders.

97. **Heavy processing in Fragment's onCreateView()**  
    **Prevent**: Defer or move processing to background threads.

98. **Expensive database queries in ContentProvider**  
    **Prevent**: Perform expensive queries asynchronously using `AsyncQueryHandler`.

99. **Running UI operations in background thread by mistake**  
    **Prevent**: Ensure UI operations are posted back to main thread using `runOnUiThread()` or `Handler`.

100. **Slow socket connection establishment**  
    **Prevent**: Use non-blocking sockets and execute them in background threads.

This comprehensive list should give you a good starting point to avoid common causes of ANRs in Android development.
