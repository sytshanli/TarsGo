# Tarsgo  Document

## About
- 	Tarsgo is high performance rpc framework in Golang programing laguage which use tars protocol.With the rise - of containerization technology such as docker, k8s, etcd, Go language has become popular. Go's coroutine concurrency mechanism makes Go is very suitable for large-scale high-concurrency back-end server program development. The Go language has near C/C++ performance and near python productivity. At Tencent , part of the existing C++ developer gradually turn into Go developer, and Tars as  widely used RPC framework, has now supported C + + / Java / Nodejs /Php, and the combination with Go language has become a general trend. Therefore, in the voice of users, we launched Tarsgo, and we have applied to Tencent map application,  YingYongbao application, Internet plus and other projects. 



## functional characteristics
- Tars2go tool: tars file is automatically generated and converted into go language, contains RPC server/client  code, implemented in go language .
Serialization and deserialization of tars in go language
- The server supports heartbeat report, stat monitoring report, custom command processing, basic log
- The client supports direct connection and route access, automatic reconnection, periodic refresh of node status, and support for UDP/TCP protocol.
- The support of remote log
- The support of property report
- The support of set division



## Install
- Require Go 1.9.x or above, see https://golang.org/doc/install
- go get -u github.com/TarsCloud/TarsGo/tars


## Quickstart
- for quickstart ,see [tars_go_quickstart_en.md](tars/docs/tars_go_quickstart_en.md)
- 快速开始，请查看 [tars_go_quickstart.md](tars/docs/tars_go_quickstart.md)


## usage
### 1 server
 - Below is a full example to illustrate how to use tarsgo to build a server.
  
#### 1.1 interface definition

write a tars file, like hello.tars ,under $GOPATH/src , like $GOPATH/TestApp/TestServer/hello.tars
	
```
	
	module TestApp
	{
	
	interface Hello
	{
	    int test();
	    int testHello(string sReq, out string sRsp);
	};
	
	}; 
	
```
	

#### 1.2 compile interface definition file

##### 1.2.1 build tars2go
compile the tars2go tools and copy tars2go binary to $PATH

	cd $GOPATH/src/github.com/TarsCloud/TarsGo/tars/tools/tars2go && go build . 
    cp tarsgo $GOPTAH/bin 

##### 1.2.2 compile the tars file and translate into go file
	tars2go --outdir=./vendor hello.tars
#### 1.3 implete the interface
```
package main

import (
    "github.com/TarsCloud/TarsGo/tars"

    "TestApp"
)

type HelloImp struct {
}

//implete the Test interface
func (imp *HelloImp) Test() (int32, error) {
    return 0, nil 
}

//implete the testHello interface

func (imp *HelloImp) TestHello(in string, out *string) (int32, error) {
    *out = in
    return 0, nil 
}


func main() { //Init servant
    imp := new(HelloImp)                                    //New Imp
    app := new(TestApp.Hello)                               //New init the A Tars
    cfg := tars.GetServerConfig()                           //Get Config File Object
    app.AddServant(imp, cfg.App+"."+cfg.Server+".HelloObj") //Register Servant
    tars.Run()
}


```

illustration:
- HelloImp is the struct  where u implete the Hello & Test interface, notice that Test & Hello must start with upper letter to be exported ,which is the only place distinguished from tars file definition. 
- TestApp.Hello is generated by tar2go tools,which could be found in ./vendor/TestApp/Hello_IF.go, with a package named TestApp which is smae as module TestApp in the tars file.
-  tars.GetServerConfig()  is used to get server side configuration.
-  cfg.App+"."+cfg.Server+".HelloObj" is the object  name bind to the Servant, which client will use this name to access the server.



#### 1.4 ServerConfig

tars.GetServerConfig()  return a server config,which is defined as below:
```
type serverConfig struct {
	Node      string
	App       string
	Server    string
	LogPath   string
	LogSize   string
	LogLevel  string
	Version   string
	LocalIP   string
	BasePath  string
	DataPath  string
	config    string
	notify    string
	log       string
	netThread int
	Adapters  map[string]adapterConfig

	Container   string
	Isdocker    bool
	Enableset   bool
	Setdivision string
}


```

Node: local tarsnode address, only if u use tars platform to deploy will use this parameter.
APP: The application name.
Server: The server name.
LogPath: The  diretory to save logs.
LogSize: The size when ratate logs.
LogLevel: The rotate log level.
Version: Tarsgo version.
LocalIP: Local ip address.
BasePath: Base path for the binary.
DataPath: Path for store some cache file.
config: The configuration center for getting configuration， like tars.tarsconfig.ConfigObj.
notify： The notify center for report notify report , like tars.tarsnotify.NotifyObj
log： The remote log center , like tars.tarslog.LogObj
netThread: Reserved  for controling the go routine that rececv and send pacakges.
Adapters:  The specify configuration for each adapters
Contianer: Reserved for later use, to store the contianer name.
Isdocker: Reserved for later use, to specify if the server runing inside a container.
Enableset: True if used set division.
Setdivision: To specify  which set division ,like gray.sz.*

A server side configuration look like:
```
<tars>
  <application>
      enableset=Y
      setdivision=gray.sz.*
    <server>
       node=tars.tarsnode.ServerObj@tcp -h 10.120.129.226 -p 19386 -t 60000
       app=TestApp
       server=HelloServer
       localip=10.120.129.226
       local=tcp -h 127.0.0.1 -p 20001 -t 3000
       basepath=/usr/local/app/tars/tarsnode/data/TestApp.HelloServer/bin/
       datapath=/usr/local/app/tars/tarsnode/data/TestApp.HelloServer/data/
       logpath=/usr/local/app/tars/app_log/
       logsize=10M
       config=tars.tarsconfig.ConfigObj
       notify=tars.tarsnotify.NotifyObj
       log=tars.tarslog.LogObj
       #timeout for deactiving , ms.
       deactivating-timeout=2000
       logLevel=DEBUG
    </server>
  </application>
</tars>

```

#### 1.5 Adapter
An adapter represent a bind ip port for centain object. 
app.AddServant(imp, cfg.App+"."+cfg.Server+".HelloObj") in the server impletment code example , fullfill the binding of 
the adapter configruation and impletment for the HelloObj.
A full example for adapter, see below:

```
<tars>
  <application>
    <server>
       #each adapter configuration 
       <TestApp.HelloServer.HelloObjAdapter>
            #allow Ip for white list.
            allow
            # ip and port to listen on  
            endpoint=tcp -h 10.120.129.226 -p 20001 -t 60000
            #handlegroup
            handlegroup=TestApp.HelloServer.HelloObjAdapter
            #max connection 
            maxconns=200000
            #portocol, only tars for now.
            protocol=tars
            #max capbility in handle queue.
            queuecap=10000
            #timeout in ms for the request in the queue.
            queuetimeout=60000
            #servant 
            servant=TestApp.HelloServer.HelloObj
            #threads in handle server side implement code. goroutine for golang.
            threads=5
       </TestApp.HelloServer.HelloObjAdapter>
    </server>
  </application>
</tars>
```


#### 1.6 start the server 

The command for starting the server：
```
./HelloServer --config=config.conf
```
See below for a full example of config.conf ,We will explain the client side configuration later.


```
<tars>
  <application>
    enableset=n
    setdivision=NULL
    <server>
       node=tars.tarsnode.ServerObj@tcp -h 10.120.129.226 -p 19386 -t 60000
       app=TestApp
       server=HelloServer
       localip=10.120.129.226
       local=tcp -h 127.0.0.1 -p 20001 -t 3000
       basepath=/usr/local/app/tars/tarsnode/data/TestApp.HelloServer/bin/
       datapath=/usr/local/app/tars/tarsnode/data/TestApp.HelloServer/data/
       logpath=/usr/local/app/tars/app_log/
       logsize=10M
       config=tars.tarsconfig.ConfigObj
       notify=tars.tarsnotify.NotifyObj
       log=tars.tarslog.LogObj
       deactivating-timeout=2000
       logLevel=DEBUG
       <TestApp.HelloServer.HelloObjAdapter>
            allow
            endpoint=tcp -h 10.120.129.226 -p 20001 -t 60000
            handlegroup=TestApp.HelloServer.HelloObjAdapter
            maxconns=200000
            protocol=tars
            queuecap=10000
            queuetimeout=60000
            servant=TestApp.HelloServer.HelloObj
            threads=5
       </TestApp.HelloServer.HelloObjAdapter>
    </server>
    <client>
       locator=tars.tarsregistry.QueryObj@tcp -h 10.120.129.226 -p 17890
       sync-invoke-timeout=3000
       async-invoke-timeout=5000
       refresh-endpoint-interval=60000
       report-interval=60000
       sample-rate=100000
       max-sample-count=50
       asyncthread=3
       modulename=TestApp.HelloServer
    </client>
  </application>
</tars>
```


### 2 client
User can write a client side code easily without writing any protocol-specified communicating code.
#### 2.1 client example
A  client side example:
```
package main

import (
    "fmt"
    "github.com/TarsCloud/TarsGo/tars"
    "TestApp"
)

func main() {
    comm := tars.NewCommunicator()
    obj := "TestApp.TestServer.HelloObj@tcp -h 127.0.0.1 -p 10015 -t 60000"
    app := new(TestApp.Hello)
    comm.StringToProxy(obj, app)
	var req string="Hello Wold"
    var res string
    ret, err := app.TestHello(req, &out)
    if err != nil {
        fmt.Println(err)
        return
    }   
    fmt.Println(ret, out)
```
illustration:
- pakage TestApp was generated by tars2go tool using the tars protocol file.
- comm: A Communicator is used for communicating  with the server side.
- obj: object name which to sepcify the server ip and port. Usally, we just need the object name before the 
"@" character.
- app: Application that associated with the intercace in the tars file. In this case, it's TestApp.Hello.
- StringToProxy: StringToProxy method is used for binding the object name and the application, if don't do this, communicator won't know who to communite with for the application.
- req, res: In and out parameter which define in the tars file for the TestHello method.
- app.TestHello is used to call the method defined in the tars file ,and ret and err will be returned .

#### 2.2 communicator
A communicator represent a group of resources for sending and receiving pacakges for the client side, which in the end manages the socket commnicating for each object.
U will only need one communicator in a program.
```
comm := tars.NewCommunicator()
comm.SetProperty("property", "tars.tarsproperty.PropertyObj")
comm.SetProperty("locator", "tars.tarsregistry.QueryObj@tcp -h ... -p ...")
```

Description:
> * The communicator's configuration file format will be described later.
> * Communicators can be configured without a configuration file, and all parameters have default values.
> * The communicator can also be initialized directly through the "SetProperty" method.
> * If you don't want a configuration file, you must set the locator parameter youself.

Communicator attribute description:
> * locator:The address of the registry service must be in the format "ip port". If you do not need the registry to locate the service, you do not need to configure this item.
> *important  async-invoke-timeout:The maximum timeout (in milliseconds) for client  calls. The default value for this configuration is 3000.
> * sync-invoke-timeout:Unused for tarsgo right now.
> * refresh-endpoint-interval:The interval (in milliseconds) for periodically accessing the registry to obtain information. The default value for this configuration is one minute.
> * stat:The address of the service is called between modules. If this item is not configured, it means that the reported data will be directly discarded.
> * property:The address that the service reports its attribute. If it is not configured, this means that the reported data is directly discarded.
> * report-interval:Unused for tarsgo for now.
> * asyncthread:  Discarded for tarsgo.
> * modulename:The module name, the default value is the name of the executable program.

The format of the communicator's configuration file is as follows:
```
<tars>
  <application>
    #The configuration required by the proxy
    <client>
        #address
        locator                     = tars.tarsregistry.QueryObj@tcp -h 127.0.0.1 -p 17890
        #The maximum timeout (in milliseconds) for synchronous calls.
        sync-invoke-timeout         = 3000
        #The maximum timeout (in milliseconds) for asynchronous calls.
        async-invoke-timeout        = 5000
        #The maximum timeout (in milliseconds) for synchronous calls.
        refresh-endpoint-interval   = 60000
        #Used for inter-module calls
        stat                        = tars.tarsstat.StatObj
        #Address used for attribute reporting
        property                    = tars.tarsproperty.PropertyObj
        #report time interval
        report-interval             = 60000
        #The number of threads that process asynchronous responses
        asyncthread                 = 3
        #The module name
        modulename                  = Test.HelloServer
    </client>
  </application>
</tars>
```
#### 2.3 Timeout control
if u want to use timeout control in the client side, use TarsSetTimeout which in ms.
```
    app := new(TestApp.Hello)
    comm.StringToProxy(obj, app)
    app.TarsSetTimeout(3000)
```

#### 2.4  Call interface

This section details how the Tars client remotely invokes the server.

First, briefly describe the addressing mode of the Tars client. Secondly, it will introduce the calling method of the client, including but not limited to one-way calling, synchronous calling, asynchronous calling, hash calling, and so on.

##### 2.4.1. Introduction to addressing mode

The addressing mode of the Tars service can usually be divided into two ways: the service name is registered in the master and the service name is not registered in the master. A master is a name server (routing server) dedicated to registering service node information.

The service name added in the name server is implemented through the operation management platform.

For services that are not registered with the master, it can be classified as direct addressing, that is, the ip address of the service provider needs to be specified before calling the service. The client needs to specify the specific address of the HelloObj object when calling the service:

that is: Test.HelloServer.HelloObj@tcp -h 127.0.0.1 -p 9985

Test.HelloServer.HelloObj: Object name

tcp:Tcp protocol

-h:Specify the host address, here is 127.0.0.1

-p:Port, here is 9985

If HelloServer is running on two servers, app is initialized as follows:
```
    obj:= "Test.HelloServer.HelloObj@tcp -h 127.0.0.1 -p 9985:tcp -h 192.168.1.1 -p 9983"
    app := new(TestApp.Hello)
    comm.StringToProxy(obj, app)
```
The address of HelloObj is set to the address of the two servers. At this point, the request will be distributed to two servers (distribution method can be specified, not introduced here). If one server is down, the request will be automatically assigned to another one, and the server will be restarted periodically.

For services registered in the master, the service is addressed based on the service name. When the client requests the service, it does not need to specify the specific address of the HelloServer, but it needs to specify the address of the `registry` when generating the communicator or initializing the communicator.

The following shows the address of the registry by setting the parameters of the communicator:
```
comm := tars.NewCommunicator()
comm.SetProperty("locator", "tars.tarsregistry.QueryObj@tcp -h ... -p ...")
```
Since the client needs to rely on the registry's address, the registry must also be fault-tolerant. The registry's fault-tolerant method is the same as above, specifying the address of the two registry.
##### 2.4.2. One-way call
TODO. Unsupported yet in tarsgo.

##### 2.4.3. Synchronous call
```
package main

import (
    "fmt"
    "github.com/TarsCloud/TarsGo/tars"
    "TestApp"
)

func main() {
    comm := tars.NewCommunicator()
    obj := "TestApp.TestServer.HelloObj@tcp -h 127.0.0.1 -p 10015 -t 60000"
    app := new(TestApp.Hello)
    comm.StringToProxy(obj, app)
	var req string="Hello Wold"
    var res string
    ret, err := app.TestHello(req, &out)
    if err != nil {
        fmt.Println(err)
        return
    }   
    fmt.Println(ret, out)

```

##### 2.4.4 Asynchronous call
tarsgo can use Asynchronous call easily using go routine. Unlike cpp, we don't need to implement a callback function.
```
package main

import (
    "fmt"
    "github.com/TarsCloud/TarsGo/tars"
    "time"
    "TestApp"
)

func main() {
    comm := tars.NewCommunicator()
    obj := "TestApp.TestServer.HelloObj@tcp -h 127.0.0.1 -p 10015 -t 60000"
    app := new(TestApp.Hello)
    comm.StringToProxy(obj, app)
	go func(){
		var req string="Hello Wold"
    	var res string
    	ret, err := app.TestHello(req, &out)
    	if err != nil {
        	fmt.Println(err)
        	return
    	} 
		fmt.Println(ret, out)
	}()
    time.Sleep(1)  

```

##### 2.4.5 call by set
Client can call Server by set through configuration file mentioned about. Which   enableset will be y and setdivision  will set like gray.sz.* . See https://github.com/TarsCloud/Tars/blob/master/docs-en/tars_idc_set.md for more detail.
If u want call by set manually, tarsgo will support this feature soon.
##### 2.4.6. Hash call
Since multiple servers can be deployed, client requests are randomly distributed to the server, but in some cases, it is desirable that certain requests are always sent to a particular server. In this case, Tars provides a simple way to achieve which is called hash-call. Tarsgo will support this feature soon.


### 3   return code defined by tars.
```
//Define the return code given by the TARS service
const int TARSSERVERSUCCESS       = 0;    //Server-side processing succeeded
const int TARSSERVERDECODEERR     = -1;   //Server-side decoding exception
const int TARSSERVERENCODEERR     = -2;   //Server-side encoding exception
const int TARSSERVERNOFUNCERR     = -3;   //There is no such function on the server side
const int TARSSERVERNOSERVANTERR  = -4;   //The server does not have the Servant object
const int TARSSERVERRESETGRID     = -5;   // server grayscale state is inconsistent
const int TARSSERVERQUEUETIMEOUT  = -6;   //server queue exceeds limit
const int TARSASYNCCALLTIMEOUT    = -7;   // Asynchronous call timeout
const int TARSINVOKETIMEOUT       = -7;   //call timeout
const int TARSPROXYCONNECTERR     = -8;   //proxy link exception
const int TARSSERVEROVERLOAD      = -9;   //Server overload, exceeding queue length
const int TARSADAPTERNULL         = -10;  //The client routing is empty, the service does not exist or all services are down.
const int TARSINVOKEBYINVALIDESET = -11;  //The client calls the set rule illegally
const int TARSCLIENTDECODEERR     = -12;  //Client decoding exception
const int TARSSERVERUNKNOWNERR    = -99;  //The server is in an abnormal position
```


### 4 log

A quick example for using tarsgo rotating log
```
TLOG := tars.GetLogger("TLOG")
TLOG.Debug("Debug logging")
```
This is will create a *Rogger.Logger ,which was defined in tars/util/rogger, and after GetLogger was called, and a logfile is created under Logpath defined in the config.conf which name is  cfg.App + "." + cfg.Server + "_" + name ,and will be rotated after 100MB(default) , and max rotated file is 10(default). 

if u don't want to rotate log by file size. For example , u want to rotate by day, then use:

```
TLOG := tars.GetDayLogger("TLOG",1)
TLOG.Debug("Debug logging")
```
for rotating by hour, use GetHourLogger("TLOG",1).
If u  want to log to remote server ,which is defined in config.conf named tars.tarslog.LogObj. A full tars file defination can be found in tars/protocol/res/LogF.tars. U have to setup  a log server before doing this. A log server can be found under Tencent/Tars/cpp/framework/LogServer .A quick  example，
```
TLOG := GetRemoteLogger("TLOG")
TLOG.Debug("Debug logging")

```
if u want to set the loglevel , u can set it from OSS platform provided by tars project under Tencent/Tars/web.
If  u want to customize ur logger， see more detail in tars/util/rogger , tars/logger.go and tars/remotelogger.go
### 5  Service management

The Tars server framework supports dynamic receiving commands to handle related business logic, such as dynamic update configuration.

tarsgo  currently has tars.viewversion / tars.setloglevel administration commands for now. User can send admin command from oss to see what version is  or setting loglevel mentioned about.

if u want to defined ur own admin commands, see this example
```
func helloAdmin(who string ) (string, error) {
	return who, nil
}
tars.RegisterAdmin("tars.helloAdmin",  helloAdmin)

```
Then u can send self-defined admin command "tars.helloAdmin  tarsgo" and tarsgo
 will be shown in browser.

Illustration:
```
// A function  should be in this format
type adminFn func(string) (string, error)

//then u should registry this function using

func RegisterAdmin(name string, fn adminFn)
```

### 6 Statistical reporting

Reporting statistics information is the logic of reporting the time-consuming information and other information to tarsstat inside the Tars framework. No user development is required. After the relevant information is correctly set during program initialization, it can be automatically reported inside the framework (including the client and the server).

After the client call the reporting interface, it is temporarily stored in memory. When it reaches a certain time point, it is reported to the tarsstat service (the default is once reporting 1 minute). We call the time gap between the two reporting time points as a statistical interval, and perform the operations such as accumulating and comparing the same key in a statistical interval.
The sample code is as follows:

```
//for error
ReportStat(msg, 0, 1, 0)

//for success
ReportStat(msg, 1, 0, 0)


//func ReportStat(msg *Message, succ int32, timeout int32, exec int32)
//see more detail in tars/statf.go
```

Description:
> * Normally, we don't have to concert about  the Statistical reporting ，the tarsgo framework will do this report 
after every client call server ，no matter success or failure. And the success rate ,fail rate ,average  cost time and so on will be shown in the web management system if u setup properly.
> *  If the main service is deployed on the web management system, you do not need to define Communicator set the configurations of tarsregistry, tarsstat, etc., the service will be automatically reported.
> * If the main service or program is not deployed on the web management system, you need to define the Communicator, set the tarsregistry, tarsstat, etc., so that you can view the service monitoring of the called service on the web management system.
> * The reported data is reported regularly and can be set in the configuration of the communicator.


### 7 Anormaly reporting
For better monitoring, the TARS framework supports reporting abnormal situdation directly to tarsnotify in the program and can be viewed on the WEB management page.

The framework provides three macros to report different kinds of exceptions:
```
tars.reportNotifyInfo("Get data from mysql error!")
```
Info is a string, which can directly report the string to tarsnotify. The reported string can be seen on the page, subsequently, we can alarm according to the reported information.



### 8 Attribute(Property) Statistics
In order to facilitate business statistics, the TARS framework also supports the display of information on the web management platform. 


The types of statistics currently supported include the following:
> * Sum(sum) //caculate the sum of every report value.
> * Average(avg) //caculate the average of every report value
> * Distribution(distr) //caculate the distribution of every report,which parameter is a list, and calculate the probability distribution of each interval
> * Maximum(max) //caculate the maximum of every report value .
> * Minimum(min) // caculate the minimum of every report value .
> * Count(count) //caculate the count of  report times

The sample code is as follows:
```
    sum := tars.NewSum()
    count := tars.NewCount()
    max := tars.NewMax()
    min := tars.NewMin()
    d := []int{10, 20, 30, 50} 
    distr := tars.NewDistr(d)
    p := tars.CreatePropertyReport("testproperty", sum, count, max, min, distr)
    for i := 0; i < 5; i++ {
        v := rand.Intn(100)
        p.Report(v)

    }   

```

Description:
> * Data is reported regularly, and can be set in the configuration of the communicator, currently once per minute;
> * Create a PropertyReportPtr function: The parameter createPropertyReport can be any collection of statistical methods, the example uses six statistical methods, usually only need to use one or two;
> * Note that when you call createPropertyReport, you must create and save the created object after the service is enabled, and then just take the object to report, do not create it each time you use.

### 9 remote configuration
User can setup remote configuration from OSS. See more detail in https://github.com/TarsCloud/TarsFramework/blob/master/docs-en/tars_config.md . 
That is an example to illustrate how to use this api to get configuration file from remote.

```
import "github.com/TarsCloud/TarsGo/tars"
...
cfg := tars.GetServerConfig()
remoteConf := tars.NewRConf(cfg.App, cfg.Server, cfg.BasePath)
config, _ := remoteConf.GetConfig("test.conf")

...

```

### 10 setting.go
setting.go in package tars  is used to control tarsgo performance and characteristics .Some option should be updated from Getserverconfig().

```
//number of woker routine to handle client request
//zero means  no contorl ,just one goroutine for a client request.
//runtime.NumCpu() usually best performance in the benchmark.
var MaxInvoke int = 0

const (
	//for now ,some option shuold update from remote config

	//version
	TarsVsersion string = "1.0.0"

	//server

	AcceptTimeout time.Duration = 500 * time.Millisecond
	//zero for not set read deadline for Conn (better  performance)
	ReadTimeout time.Duration = 0 * time.Millisecond
	//zero for not set write deadline for Conn (better performance)
	WriteTimeout time.Duration = 0 * time.Millisecond
	//zero for not set deadline for invoke user interface (better performance)
	HandleTimeout  time.Duration = 0 * time.Millisecond
	IdleTimeout    time.Duration = 600000 * time.Millisecond
	ZombileTimeout time.Duration = time.Second * 10
	QueueCap       int           = 10000000

	//client
	ClientQueueLen     int           = 10000
	ClientIdleTimeout  time.Duration = time.Second * 600
	ClientReadTimeout  time.Duration = time.Millisecond * 100
	ClientWriteTimeout time.Duration = time.Millisecond * 3000
	ReqDefaultTimeout  int32         = 3000
	ObjQueueMax        int32         = 10000

	//report
	PropertyReportInterval time.Duration = 10 * time.Second
	StatReportInterval     time.Duration = 10 * time.Second

	//mainloop
	MainLoopTicker time.Duration = 10 * time.Second

	//adapter
	AdapterProxyTicker     time.Duration = 10 * time.Second
	AdapterProxyResetCount int           = 5

	//communicator default ,update from remote config
	refreshEndpointInterval int = 60000
	reportInterval          int = 10000
	AsyncInvokeTimeout      int = 3000

	//tcp network config
	TCPReadBuffer  = 128 * 1024 * 1024
	TCPWriteBuffer = 128 * 1024 * 1024
	TCPNoDelay     = false
)


```