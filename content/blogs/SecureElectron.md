---
title: "SecureElectron"
date: 2021-04-09
weight: 1
aliases: [""]
tags: ["electronjs","security"]
author: ["Tapan Manu","Sujith VI","Sreejith A"]
categories: [""]
series: [""]
description: "A study of secure sandboxes with Electron JS"
summary: "A study of secure sandboxes with Electron JS"
searchHidden: false
# cover:
#     image: "<image path/url>" # image path/url
#     alt: "<alt text>" # alt text
#     caption: "<text>" # display caption under cover
#     relative: false # when using page bundles set this to true
#     hidden: true # only hide on current single page
# editPost:
#     URL: "https://github.com/<path_to_repo>/content"
#     Text: "Suggest Changes" # edit text
#     appendFilePath: true # to append file path to Edit link
---

# Introduction

Electron is one of the most popular frameworks in Java script for desktop applications. It’s popularity is mainly due to the NodeJS integration it has. The time consuming process for every application is once when they access host system resources. In addition to that, Electron applications may be prone to security vulnerabilities due to which it can directly run third party modules into application context being unverified.  Hence, a sandboxing mechanism is required for context isolation and running third party modules without much degradation in the performance it imparted earlier. Moreover, this paper focuses on some techniques that can be made in the sandbox global object for verifying the third party modules and libraries by incorporating customisation. Moreover, it also deals with several experiments to measure the performance of electron apps in main context, context provided within web workers and in sandboxed context.
Keywords to be considered:
* Nodejs Integration
* VM2 Sandbox
* Electron Security analysis 
* File Read performance.


# ARCHITECTURE AND FEATURES

Electron consists of 2 processes, Main and Renderer. The Main process manages all web pages and their corresponding Renderer processes. It creates web pages by creating BrowserWindow instances. Each of these instances runs the web page in its Renderer process. When this instance is destroyed, the corresponding Renderer process also gets terminated. The independently running Renderer process manages only the corresponding web page. Direct calls to GUI are restricted due to security concerns. The GUI operations are performed by the Renderer process via IPC (Inter-Process-Communication). The modules for Inter-Process-Communication (IPC) are, _ipcMain_ and _ipcRenderer_ 

![Architecture](https://user-images.githubusercontent.com/56436219/111844130-68825080-8928-11eb-8f37-403da76eb564.png)

![Architecture](https://user-images.githubusercontent.com/56436219/111844404-ffe7a380-8928-11eb-8bcc-34fa2cedb19f.png)

The architecture can be described using views and perspectives also. The view is a structural aspect of architecture and perspective is a collection of activities, and how a system behaves. The views are further divided into context, deployment, and development. The context view describes the scope and responsibilities, relationships, dependencies, and interactions around Electron with the users and frameworks. The development viewpoint is the layered modular architecture of electrons which includes the processes, APIs, and design models. The deployment viewpoint is the view of the dependencies which make Electron work. This view summarizes the system requirements and additional frameworks to successfully run Electron. The perspective deals with the evolution of Electron as well as the security features incorporated in this. 

**Nodejs integration** 

Nodejs is a non-blocking IO model that is event-driven and lightweight, which is suitable for data-intensive applications that run on distributed devices. Electron supports custom APIs for working with native OS functions. There are several APIs available in the Main process and the Renderer process accessible by its included module. Electron APIs are assigned based on the process type, meaning that some modules can be used from either the Main or Renderer process, and some from both. Electron exposes full access to Node.js API and its modules both in the Main and the Renderer processes. Renderer processes may run untrusted code (especially from third parties), it is important to carefully validate the requests that come to the Main process. Node APIs also have a disadvantage that it is not stable as it keeps on changing frequently. At times, a new API appears to have many backward-incompatible changes. As a result, the developers are forced to make changes in the accessible code bases to match the compatibility with the latest version of the Node.js API.

The advantage of using Node runtime as its back-end component as it interacts with the native Operating System very smoothly. Electron and its two dependencies use Google's V8 high-performance JavaScript engine which adds up as a runtime environment. Moreover, it supports cross-platform includes 32bit and 64bit OS. 

**Binding Node and Electron**

In regards to how Electron handles the JavaScript contexts of Node.js and Chromium, it keeps the backend code’s JavaScript state separate from that of the frontend application windows’ state. This isolation of the JavaScript state is one of the ways that Electron is different from NW.js. That said, Node.js modules can be referenced and used from the frontend code as well, but it’s operating in a separate process to the backend. This explains why data sharing between the backend and application windows is handled via inter-process communication or “message passing” as it’s also called.

Enabling Node.js integration in Electron web content renderers (BrowserWindow, BrowserView, and review) can result in remote native code execution attacks. The attack is realized when the renderer uses content from an untrusted remote web site or a trusted site with a cross-site scripting vulnerability. Node.js integration should be disabled when loading remote web sites, thus limiting the powers granted to remote content. It is advised to set nodeIntegration preference to false before loading remote web sites, and only enable it for whitelisted sites with protocols HTTPS.
The security of applications built on Electron depends on the current, latest version of Chromium. Chromium has an off-the-shelf sandbox to run processes that can freely use CPU and memory. However, being a restrictive environment, there are well-defined policies for processes that prevent bugs and attacks during IO operations. To provide applications with desktop functionality, the Electron team modified Chromium to introduce a runtime to access native APIs as well as Node.js built-in and third-party modules. To do this the Chromium sandbox protection was disabled, meaning any application running inside Electron is given unfiltered access to the operating system. This is a (potentially) serious and unnecessary security risk for applications that require remote data, any successful XSS attack would give the attacker full control over the victim’s machine. Hence, there is an urgent need to develop a secure sandbox.

Protocols like HTTPS, WSS, or FTPS are recommended for loading external resources into our application due to data encryption, data integrity, and ensuring the correct host. Receiving code from an untrusted domain always poses a security issue and executing it locally. An attacker might be able to execute native code on the user's machine if node integration is enabled. A sample code to illustrate the scenario is provided in Appendix A.

A cross-site-scripting (XSS) attack is more dangerous if an attacker can jump out of the renderer process and execute code on the user's computer. Cross-site-scripting attacks are fairly common - and while an issue, their power is usually limited to messing with the website that they are executed on. Disabling Node.js integration helps prevent an XSS from being escalated into a so-called "Remote Code Execution" (RCE) attack. Even restricted by the sandbox, an attacker can execute scripts to hijack user sessions, deface web sites, insert hostile content, redirect users, and install malware. But the potential for damage stops at the browser level with the operating system left unharmed and only data explicitly shared with the browser at risk.
A malicious preload script can still obtain a reference to the application module by leveraging the remote module, which provides a simple way to do Inter-Process Communication (IPC) between the renderer and main processes. Alternatively, it is also possible to emulate the internal IPC mechanism sending a message to the main process synchronously via ELECTRON_ internal channels. Considering privileged access in preload, the code of preload scripts need to be carefully reviewed. [5]

**Context Isolation**

To display remote content, disable nodeIntegration, and enable context Isolation. Context isolation is an Electron feature that allows developers to run code in preload scripts and Electron APIs in a dedicated JavaScript context. It's not affected by any changes, malicious or otherwise, that a page can make to JavaScript globals or object prototypes.In practice, that means that global objects like Array.prototype.push or JSON.parse cannot be modified by scripts running in the renderer process.

Experiment 1

The entry point of the given experiment was considered as rd.js (Appendix C, code example 1) which was considered to be the entry file in Electron application by modifying the package.json file.

The procedure can be listed :
* Set the current javascript file as entry point by modifying package.json 
* Create files of various sizes ranging from 1KB to 1GB where each file is named as filex.txt where x denotes the number ranging from 1 to 7. I.e, file1.txt corresponds to a file of size 1KB and file7.txt corresponds to 1 GB size file.( The files are generated with a shell script that utilizes the truncate command to generate dummy files of specified size ). Code is provided in Appendix I, snippet 1.
* Compute the DOMHighResTimeStamp using performance.now() and assign it as t1.
* Perform synchronous file access for file1.txt using readFileSync() and compute the time t2 by performance.now() available in perf_hooks module .
* Compute t2-t1.
* Repeat the steps 3 to 5 for asynchronous access (readFile() command) for file1.txt.
* Repeat steps 3 to 6 for file2.txt, file3.txt, …. , file7.txt and note down the observations in a table.

![Table A](https://user-images.githubusercontent.com/56436219/111844658-797f9180-8929-11eb-9907-eb0fb1f8c50d.png)

![Graph A](https://user-images.githubusercontent.com/56436219/111844917-e7c45400-8929-11eb-8f9f-7863899848cb.png)

Observed result -Experiment 1

Two results can be drawn from this experiment:
* The time taken for  synchronous file access is 97% lesser than asynchronous file access of same sized files of size less than or equal to 1MB and 74% lesser than files greater than 10MB .
* The average increase in file access time was 4.29 ms for each 1MB increase of data for asynchronous file access and  1.30 ms per MB for synchronous access.

Experiment 2

The experiment followed similar procedures as that of experiment 1. Here, both synchronous and asynchronous directory access operations are performed. The directories considered were single level directories with varying numbers of files. (ranging with 100, 1000 and 10000 files).


The procedure can be listed :
* Set the current javascript file as entry point by modifying package.json 
* Create a single level directory testdir consisting of files of different sizes and numbers .
* Code is provided in Appendix I, snippet 2.
* Compute the DOMHighResTimeStamp using performance.now() and assign it as t1.
* Perform synchronous file access for file1.txt using readdirSync() and compute the time t2 by performance.now() available in perf_hooks module .
* Compute t2-t1.
* Repeat the steps 3 to 5 for asynchronous access (readdir() command) for file1.txt.
* Repeat steps 3 to 6 for different directories note down the observations in a table.

![Table B](https://user-images.githubusercontent.com/56436219/111845108-3ffb5600-892a-11eb-9020-222208aa9d65.png)

![Graph B](https://user-images.githubusercontent.com/56436219/111845115-438edd00-892a-11eb-8e21-6b9fe07af610.png)

The conclusions that can be drawn are:
The synchronous single level directory access is  98.6 % lesser than asynchronous directory access.


Experiment 3
Study on OS Caching effect in single file access in electron. This was conducted in tandem with that of experiment 1. Here, the effect on file access time without OS Caching and with OS Caching is analysed and compared. The dummy files created during experiment 1 were also considered.

Procedure:
* Set the current javascript file as entry point by modifying package.json 
* Create files of various sizes ranging from 1KB to 1GB where each file is named as filex.txt where x denotes the number ranging from 1 to 7. I.e, file1.txt corresponds to a file of size 1KB and file7.txt corresponds to 1 GB size file.( The files are generated with a shell script that utilizes the truncate command to generate dummy files of specified size ). Code is provided in Appendix I, snippet 1.
* Compute the DOMHighResTimeStamp using performance.now() and assign it as t1.
* Perform synchronous file access for file1.txt using readFileSync() and compute the time t2 by performance.now() available in perf_hooks module .
* Compute t2-t1.
* Repeat steps 3 to 5 again for the same file 2 or 3 times and observe the difference in access time.
* Repeat the steps 3 to 5 for asynchronous access (readFile() command) for file1.txt.
* Repeat steps 3 to 6 for file2.txt, file3.txt, …. , file7.txt and note down the observations in a table.

![Table C](https://user-images.githubusercontent.com/56436219/111845317-9bc5df00-892a-11eb-8a7c-e3ff5210d6a3.png)

![Graph C](https://user-images.githubusercontent.com/56436219/111845383-b8faad80-892a-11eb-9fbc-963ea0e9f4ee.png)

Observed Results - Experiment 3

For asynchronous transfers, caching effect is observed at an average of 16.9 % reduction and 23.8% reduction for synchronous transfers. 

The conclusions that can be drawn from the above three experiments indicate that large file accesses (regardless of synchronous or asynchronous access) depend on the OS level threads, system configuration, caching. Hence the exact values may vary from system to system. But an average analysis can be derived on the basis of this.

# DATA TRANSFER MECHANISMS

**Synchronous transfer mechanisms and  performance analysis**

Enabling node js integration allows access to node modules like to require easily, which can therefore be used for calling or analyzing web worker performance. Inorder to demonstrate the synchronous transfer mechanism, Experiment 4 is conducted. The code corresponding to this is provided in Appendix A and Appendix E.

**Experiment 4**

The given experiment aims to conduct the performance analysis test of data passing between main javascript file and worker script ( running in another thread ) via postMessage(). As similar to experiment 1, dummy files of file-x format are created (x indicates the number ranging from 1 to 7) of different sizes, and here, synchronous file access is performed on the worker thread rather than main thread (synchronous file access in webworker.js file). The time required for the given operation is analysed using the Performance module in Electron Js. The time thus determined after the operation is sent back to the main thread as a callback. Here, dedicated web workers are being used. Table D indicates the results obtained for different sized files. Here, minimum and maximum time taken for the operation is provided as end points. The reason for neither being choosing a specific value nor an average value is since the file read operation is conducted in worker thread, there is no guarantee when would the worker thread be invoked, and thus cannot exactly determine the time taken for the web worker call (since first checkpoint for performance.now() function before calling web worker was taken in main thread, the execution of web worker determines the time taken to complete the operation). Hence, varying values of time are returned. The returned values were displaying more randomness in results and hence mean was not considered. But it was observed that the values were clipped between two intervals of time, hence followed  minimum and maximum values. During the experiment it was also observed that file transfers of large sized files (500 MB, 1GB ) resulted in slow run of application and almost halted the system. An error was also generated regarding buffer size limitation of huge file transfers.  Hence the results of 500 MB, 1 GB file transfers are omitted in the result. The exact error generated is provided along with the code in Appendix E for reference. 

performance.now() is the function that returns DOMHighResTimeStamp in millisecond.

Procedure:
Set the current javascript file as entry point by modifying package.json 
Create files of various sizes ranging from 1KB to 1GB where each file is named as filex.txt where x denotes the number ranging from 1 to 7. I.e, file1.txt corresponds to a file of size 1KB and file7.txt corresponds to 1 GB size file.( The files are generated with a shell script that utilizes the truncate command to generate dummy files of specified size ). Code is provided in Appendix I, snippet 1.
Compute the DOMHighResTimeStamp using performance.now() and assign it as t1.
Create a webworker.js file and proceed step 5 to 6 by passing t1 via postMessage()
Perform synchronous file access for file1.txt using readFileSync() and compute the time t2 by performance.now() available in perf_hooks module .
Compute t2-t1 as time.
Pass the time back to main thread via postMessage()
Repeat steps 3 to 9 for file2.txt, file3.txt, …. , file7.txt and note down the observations in a table.

![Table D](https://user-images.githubusercontent.com/56436219/111845799-94eb9c00-892b-11eb-8a5e-bb365dee26a5.png)

![Graph D](https://user-images.githubusercontent.com/56436219/111845806-974df600-892b-11eb-8052-d6584d355fcd.png)

**Async transfer mechanisms, performance analysis**

Asynchronous file transfer mechanism study can be evaluated with the help of experiment 5 in  which procedure is in the same way as that of experiment 4. But the only difference that makes is the use of asynchronous file access commands rather than synchronous in experiment 4.

**Experiment 5**

The experiment is conducted in tandem with experiment 4 , only difference is the use of asynchronous file access. The table format is also chosen similar to that of experiment 4 due to the same observations as that of previous.
The procedure is the same , exception comes only when asynchronous file access is considered.

![Table E](https://user-images.githubusercontent.com/56436219/111845984-f6ac0600-892b-11eb-8f7f-eb4056230c6d.png)

![Graph E](https://user-images.githubusercontent.com/56436219/111845989-f875c980-892b-11eb-8edb-a282c6bf170f.png)

**Result**

The result observed was the time taken for file access in worker threads were also proportional to the size of file.  A peculiar relation cannot be defined correlating size and time taken to access files since the time of action of OS level thread along with worker threads are unpredictable, and the result will always depend upon interleaving operations of multiple threads (including daemon threads) running in background. 

# PERFORMANCE

Performance analysis of fs API in Electron

The fs module enables interacting with the file system on the device. [6] After a user grants a web app access, the API allows them to read or save changes directly to files and folders on the user's device. Beyond reading and writing files, the File System Access API provides the ability to open a directory and enumerate its contents. Since node integration is enabled, Electron can use the fs API for file access. But, it is necessary to evaluate the performance of file access of different sizes ranging from 1KB to 1GB  inorder to ensure Electron does not degrade in its performance, even accessing the largest file.  Three experiments are required to draw the conclusion.
Experiment 1 deals with synchronous and asynchronous file accessing operations in Electron of different sized files from 1KB to 1 GB and their comparison.
Experiment 2 deals with synchronous and asynchronous directory access operations with different numbers of  files in it. 
Experiment 3 deals with OS Caching effect in single level directory access. 

# SANDBOX


Sandboxing is a powerful technique for reducing the risk that a clever attacker will be able to exploit holes in your own code. By separating a monolithic application into a set of sandboxed services, each responsible for a small chunk of self-contained functionality, attackers will be forced to not only compromise specific frames’ content, but also their controller.

Different types of sandboxes considered here:

A. Jailed Sandbox

It is a deprecated sandboxing technique and is a small JavaScript library for running untrusted code in a sandbox. The library is written in vanilla-js and has no dependencies. It is one of the flexible JS sandboxes which can  load untrusted code in the sandbox, export a set of functions
The main application can be interacted with the untrusted code directly, but the functions to be exported is determined by the application owner, hence the operations allowed for untrusted code. 
The code is executed as a plugin, a special instance running as a restricted subprocess (in Node.js), or in a web-worker inside a sandboxed frame (in case of a web-browser environment). The iframe is created locally so that you don't need to host it on a separate (sub)domain.

B. V8 Sandbox

This module implements an isolated JavaScript environment that can be used to run any code without being able to escape the sandbox. The V8 context is initialized and executed entirely from C++ so it's impossible for the JS stack frames to lead back to the nodejs environment. It's usable from a nodejs process, but the JS environment is pure V8. The sandboxed V8 context is executed from a separate nodejs process to enable full support for script timeouts.
V8 sandboxing provides an isolated sandbox for executing any third party js code. Hence it blocks or prevents any unverified code from executing in the main context. Consequently, asynchronous function support is also enabled.
It exposes nodejs (host) functionality selectively using require parameter new Sandbox({ require: 'path-to-file.js' }). This file will be required by the workers and define both synchronous and asynchronous functions that can be called from the sandbox. Note that the native functions cannot be directly exposed, all input and output from the native functions is serialized with JSON between native sandboxes.

# WEBWORKER ANALYSIS


Javascript is a single-threaded programming language. All the processing tasks are allowed to be done within the main thread itself. If any of them is a heavy computationally intensive task, this might even slow down the application being run, consequently the browser. Thus it is not possible to develop GUI intensive applications directly using this model. Web workers are the solution to this. They are in-browser threads that can be used to execute JavaScript code without blocking the event loop. Hence allowing developers to put long-running and computationally intensive tasks in the background without blocking the UI, making your app even more responsive. There is an attempt to bring multi-threaded behavior to JavaScript. The data sent between the main thread and the workers are copied rather than shared.

The recent HTML5 standard includes the Web Workers API that allows executing JavaScript applications on multiple threads or workers. The internals of the browser’s JavaScript virtual machine does not expose direct relation between workers and running threads in the browser and the utilization of logical cores in the processor.


The web workers can be initiated by passing the path of external javascript files as argument to the web worker constructor. Worker("path/to/worker/script"). This also works for Blob URLs. It inherits properties from its parent, EventTarget, and implements properties from AbstractWorker.
GenericWebWorker enables the execution of javascript WebWorkers without using external js files, this WebWorkers allow the user to:
run javascript blocking code without stopping the main thread
pass a generic function to be executed.
pass functions to be executed in the WebWorker.
It specifies dynamic code execution within workers without requiring js files. Moreover error-catch mechanism is also implemented. exec() function is responsible for executing the required code, hence any highly math-intensive operations can be provided with generic workers without blocking the main thread.
The main idea behind the generic workers is to allow the same application programming interface (API) to be used for programming both parallel and distributed applications.
During its creation, a generic worker is specified to be either local or remote. Except for this, the same generic worker code can be executed without any changes on any platform, locally inside a browser or remotely on a server, which we call a compute server.

**Web workers management and message passing mechanism**

Web developers are often required to implement different versions of their applications to support both high-end and low-end devices. Based on existing web standards, a new web application framework that allows web worker processes to be offloaded from weak clients to powerful servers in a seamless manner can be proposed. As a result, developers can simply create applications that target high-end devices and expect the framework to help low-end devices run their applications through process offloading.
The Worker interface of the Web Workers API represents a background task that can be created via script, which can send messages back to its creator. 
Message passing mechanism of workers can be modelled as Data is sent between workers and the main thread via a system of messages. Both sides send their messages using the postMessage() method, and respond to messages via the onmessage event handler (the message is contained within the Message event's data attribute). The data is copied rather than shared. Workers may themselves spawn new workers, as long as those workers are hosted at the same origin as the parent page.
When a runtime error occurs in the worker, its onerror event handler is called. It receives an event named error which implements the ErrorEvent interface. The event doesn't bubble and is cancelable; to prevent the default action from taking place, the worker can call the error event's preventDefault() method.

**Types of Web Workers**

**Dedicated workers**

A dedicated web worker is only accessible by the script that called it.Generally it would be the main process and can only communicate with it. Worker() constructor needs to be invoked, specifying the URI of a script to execute in the worker thread such that 
If you need to immediately terminate a running worker from the main thread, you can do so by calling the worker's terminate method. The worker thread is killed immediately.

**Shared workers**

A shared worker is accessible by multiple scripts  even if they are being accessed by different windows, iframes, or even workers. Hence can be reached by all processes running on the same origin.
Creating a shared worker is very similar to how to create a dedicated one, but instead of the straight-forward communication between the main thread and the worker thread, you'll have to communicate via a port object, i.e., an explicit port has to be opened so multiple scripts can use it to communicate with the shared worker.
Spawning a shared worker : Spawning a new shared worker is pretty much the same as with a dedicated worker, but with a different constructor name, each one has to spin up the worker using code as attached in Appendix D.
One big difference is that with a shared worker you have to communicate via a port object i.e, an explicit port is opened that the scripts can use to communicate with the worker (this is done implicitly in the case of dedicated workers).
A port object is needed to invoke postMessage() function in shared workers.

**Service Worker** 

A Service Worker is an event-driven worker registered against an origin and a path. It can control the web page/site it is associated with, intercepting and modifying the navigation and resource requests, and caching resources in a very granular fashion to give you great control over how your app behaves in certain situations (e.g. when the network is not available.)

C. VM2 Sandbox

VM2 Sandbox  is a sandbox that runs untrusted code with whitelisted node modules. Limiting the access to process’ methods helps to run untrusted code securely in a single process with the code side by side. Consequently, certain built-in modules are restricted. For external modules, the sandbox needs to require those, this can be done by overriding the built-in require module. Vm2 often has an internal VM module that creates a secure context, thus immune to all kinds of attacks. The isolated scope of Vm2 prevents vulnerabilities, keeping the sandbox away from the failure of dependencies if any bugs expose the fs object, node’s global object. It uses proxies to prevent escaping the sandbox. Proxies can be thought of as decorators for objects and functions. The methods can be called securely and data and callbacks can be exchanged between sandboxes.

# VM2 ANALYSIS


Even Though vm2 provides a restriction to fs modules, overriding the built-in require modules allows external file transfers. The given section deals with transfer of different sized files and analyzes the performance of vm2 sandbox in electron. The console attribute by default is inherit, which enables the console and sandbox attribute is the global object.

**Experiment 6**

The aim of the experiment is to analyse the performance of file access in electron when vm2 sandboxing is enabled. This is to ensure sandboxing or creating a separate context from main does not degrade the performance of Electron applications. 
The procedure is detailed as:
* Define the sandbox global object.
* Allow fs and perf_hooks by overriding the require. 
* Define fread() within the sandbox object for synchronous file read.
* Create filex of specified size by using truncate command where x denotes number from 1 to 7. (each file corresponds to different sizes). Initially consider file1 which is 1KB file.
* Execute the code within vm.run() context.
* Observe the results and repeat the steps 4 and 5  for 10KB,100KB,....,100MB,1GB files.

![Table F](https://user-images.githubusercontent.com/56436219/111846329-8c479580-892c-11eb-9d1f-d571d27ad6c3.png)

![Graph F](https://user-images.githubusercontent.com/56436219/111846332-9073b300-892c-11eb-89ef-c6ee7df583f0.png)

**Result**

The time taken for file access is proportional to the size of the file. It was approximately 
0.96 ms/MB for file size between 1 and 10 MB, 1.296 ms/MB in between 10MB and 100MB,
and 6.2983 ms/MB between 100 MB and 1 GB. The reason for the variation is due to the influence of context switching between main and sandbox contexts. For large file transfers, the context switching is more frequent, therefore the probability is even more for increase in access time.

**Experiment 7**

The aim of the experiment is to analyse the performance of data transfer between Electron main context and sandbox context. This is to ensure sandboxing or creating a separate context from main does not degrade the performance of Electron applications. 
The procedure is detailed as:
* Define the sandbox context
* Allow fs and perf_hooks by overwriting the built-in module.
* Create test object string array and perform data transfer within vm2 context.
* Observe the results and repeat step 4 for different test objects like string arrays,integer arrays, variables.

**Data transfer between the main process and VM**
* String array of length 3
* String array of length 500
* Integer array of length 500
* Javascript Objects
* Javascript functions

![Table G](https://user-images.githubusercontent.com/56436219/111846463-e183a700-892c-11eb-963a-ed440b1d2078.png)

**Result**

The given experiment has listed out the time required to transfer different objects. Even though there were size differences in objects, as well as different objects were used, there was no considerable difference in time, only change was observed in transferring functions possibly due to more frequent context switching that has taken place.



# CUSTOMISED SANDBOX


Sandboxing is an essential factor which is capable of blocking any external APIs that could expose the data to the outside environment. For frameworks like electron, since it allows integration of Nodejs, any external third party modules can be incorporated easily. There is no proper verification of the code before it’s use. Thus, the need for a sandbox.
VM2 Sandbox can be modified to restrict the access of node APIs by overriding the require module. For external modules to be whitelisted, it needs to be specified in the external field of sandbox.


**Customising Sandbox with URL Access**

The experiment was done as a part to modify the existing sandbox object in such a way to block specific URLs or sites from particular domains. The sandbox object is defined and an array list called allowed_dom is created which permits the access only to allowed domains.  
The built-in modules are overwritten to allow url API to be accessed within sandbox context such that URL parsing can be done. URL parsing is allowed via Node’s url module. access_dom() is the function which is used to verify allowed domains and allows to perform URL access only to the specified domains. This is analogous to the whitelisting process of sandbox. Here, specific domains are allowed to be whitelisted and domains not specified within the allowed_dom object  have denied access. For all other domains, the sandbox throws an exception regarding restricted access. It is also possible to allow all domain access by specifying the asterisk(*) symbol within allowed_dom. Any processes that are allowed to run within the sandbox context can therefore access URLs of specific domains easily. 
Here, the code allows to specify that URL parsing is allowed to perform within the sandbox context only. 

```javascript
const {VM, NodeVM} = require("vm2");`
const { performance,PerformanceObserver } = require("perf_hooks");`
 
let ext = {`
   // defining a customised sandbox`
   allowed_dom:["stackoverflow.com"],  //specifying * allows all domains
   verify_dom(domain_name){
       return this.allowed_dom.includes(domain_name)|| this.allowed_dom[0]==="*";
   },
   access_dom(domain_name,request,url){
       if(this.verify_dom(domain_name)){
           let t1 = performance.now();
           let q = request.parse(url,true);
          let t2 = performance.now();`
           console.log(t2-t1);`
       }
       else{
           console.log("access denied");
       }
  }  
};
const vm = new NodeVM( {
   console: "inherit",
   // pass our declared ext variable to the sandbox
   sandbox: { ext },
   require: {
     external: true,
     builtin: ["url","perf_hooks"],
     root: "./",
   },
 } );
try{
   //defining whether it is part of allowed domains
   vm.run(
      const request = require("url");
       const {performance} = require("perf_hooks");
       const url="https://stackoverflow.com/questions/38811259/how-to-access-a-url-using-node-js";
       const host="github.com";
       try{
           ext.access_host(host,request,url);
       }
       catch(err){
           console.log(err);
       }
   );
   //performance of URL parse returned 3.28 ms
}
catch(err){
   console.error("Failed to execute script.", err);
}
```

**Customising Sandbox with file access**

The experiment was done in tandem with the previous experiment. Here, the aim of the experiment is to modify the existing sandbox to allow only specific file accesses, while blocking the rest. Here, the whitelisted files are specified in allowed_path object. The architecture of the customised sandbox is similar as that of the first case. The fs API is whitelisted by overriding built-in require and verify_path() is used to specify only whitelisted modules are being accessed in the sandbox context. For all other paths, the sandbox throws an exception regarding restricted access. It is also possible to allow all path access by specifying the asterisk(*) symbol within allowed_path. Any processes that are allowed to run within the sandbox context can therefore access files of specific paths  easily.  

```javascript
const {VM, NodeVM} = require("vm2");
const { performance} = require("perf_hooks");
const fs  = require("fs");
const path = require("path");
const globs = "/";
 
// By providing a file name as second argument you enable breakpoints`
 
let ext = {
 // defining a customised sandbox
 
   allowed_paths:[globs],  //specifying * allows all paths
   isChildOf(child, parent){
     if (child === parent) return false
    const parentTokens = parent.toString().split(path.sep).filter(i => i.length)
     return parentTokens.every((t, i) => child.split(path.sep)[i] === t)
   },
  
  
   verify_path(pathname){
       return this.allowed_paths.includes(pathname)|| this.allowed_paths[0]==="*";
   },
   access_path(pathname,filename){
       if(this.verify_path(pathname)){
           let t1 = performance.now();
          fs.readFile(filename,(error,data)=>{
             if(error){
              console.log(error);
              return;
           }
           let t2 = performance.now();
           console.log(t2-t1);
           });
       }
       else{
           console.log("access denied");
       }
   }  
 
 
};
 
const vm = new NodeVM( {
   console: "inherit"
  // pass our declared ext variable to the sandbox
   sandbox: { ext },
   require: {
     external: true,
     builtin: ["fs", "path"],
     root: "./",`
   },
 } );
try{
   vm.run(
       const fs = require("fs");
       const path = require("path");
       let file =process.cwd()+"demo/file4.txt";
       let root = "/";
       ext.access_path(root,file);
   )
}
catch(err){
   console.error("Failed to execute script.", err);
}

```

# CONCLUSION


At present, the technologies for providing a better sandbox for Electron is still in progress. To implement the program, the Electron runtime was chosen, which allows creating cross-platform applications based on native javascript.  There is no  right or wrong answer when it comes to the selection of build tools for next projects. Electron js provided a similar performance in file access when the operation was performed in electron main context as well as in vm2 sandbox context (except for the 1GB file which had more access time in sandboxed context ). The VM2 Sandbox has performed well when compared to that of web workers. Here,
File access within web workers had even more time taken, the buffers could not handle large file sizes (for 1GB files) , OS Caching effect could not be observed since the frequent context switching between threads compared to main context and even halted the application for 1GB file transfers.
VM2 Sandbox had better support as it allowed electron application to retain its performance in accessing external files or libraries very conveniently, without much degradation in its performance. Moreover, the sandbox can also be modified in order to allow or verify specific domains or paths. This is similar to that of whitelisting procedure. Hence any external third party modules can be allowed if it is verified to be harm-free. Thereby, with a customized sandbox Electron can overcome its one of the main disadvantages in security, without compromising in its performance. 
The distinctive features of the system that can be identified are: A separate runtime context for executing whitelisted modules without affecting the electron main context, Internal VM Support, and Security.


**Acknowledgement**

We thank Arun Bose and Charly P Abraham of Core.ai private limited for introducing us towards Electron JS Framework, and guiding us towards exploring the possibility of Security analysis in the framework. We also offer our sincere gratitude towards Dr.Ansamma John, Professor and HOD, Department of Computer Science and Engineering, TKM College of Engineering for providing guidance towards publishing the paper. 

**Authors:**

Tapan Manu, 5th Semester student, Comp Science and Engineering, TKM College of Engineering

Sujith VI, 5th Semester student, Comp Science and Engineering, TKM College of Engineering

Sreejith A, 5th Semester student, Comp Science and Engineering, TKM College of Engineering






