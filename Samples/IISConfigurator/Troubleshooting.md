# IISConfigurator.POC Troubleshooting

## Disclaimer
This is a prototype application. 
We do not recommend using this on your production environments.

- [Quick Start Instructions](QuickStart.md)
- [Detailed Instructions](DetailedInstructions.md)

## Known Issues

### Conflicting DLLs in an application's bin directory

If any of these dlls are present in the bin directory, attach may fail.

- Microsoft.ApplicationInsights.dll
- Microsoft.AspNet.TelemetryCorrelation.dll
- System.Diagnostics.DiagnosticSource.dll

Some of these are included in Visual Studio's default application templates, even if your application doesn't use them.
Sympomatic behavior can be seen using troubleshooting tools:

- PerfView:
	```
	ThreadID="7,500" 
	ProcessorNumber="0" 
	msg="Found 'System.Diagnostics.DiagnosticSource, Version=4.0.2.1, Culture=neutral, PublicKeyToken=cc7b13ffcd2ddd51' assembly, skipping attaching redfield binaries" 
	ExtVer="2.8.13.5972" 
	SubscriptionId="" 
	AppName="" 
	FormattedMessage="Found 'System.Diagnostics.DiagnosticSource, Version=4.0.2.1, Culture=neutral, PublicKeyToken=cc7b13ffcd2ddd51' assembly, skipping attaching redfield binaries" 
	```

- iisreset + app load (NO TELEMETRY). Investigate with Sysinternals (Handle.exe and ListDLLs.exe)
	```
	.\handle64.exe -p w3wp | findstr /I "InstrumentationEngine AI. ApplicationInsights"
	E54: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\Microsoft.ApplicationInsights.IISConfigurator.POC\content\Runtime\Microsoft.ApplicationInsights.RedfieldIISModule.dll

	.\Listdlls64.exe w3wp | findstr /I "InstrumentationEngine AI ApplicationInsights"
	0x0000000009be0000  0x127000  C:\Program Files\WindowsPowerShell\Modules\Microsoft.ApplicationInsights.IISConfigurator.POC\content\Instrumentation64\MicrosoftInstrumentationEngine_x64.dll
	0x0000000009b90000  0x4f000   C:\Program Files\WindowsPowerShell\Modules\Microsoft.ApplicationInsights.IISConfigurator.POC\content\Instrumentation64\Microsoft.ApplicationInsights.ExtensionsHost_x64.dll
	0x0000000004d20000  0xb2000   C:\Program Files\WindowsPowerShell\Modules\Microsoft.ApplicationInsights.IISConfigurator.POC\content\Instrumentation64\Microsoft.ApplicationInsights.Extensions.Base_x64.dll
	```

## Troubleshooting
	
### Troubleshooting PowerShell

Can audit installed Modules using cmd: `Get-Module -ListAvailable`


### Troubleshooting PowerShell Module


- If the Module has not been loaded into a PowerShell session, can manually load using the command `Import-Module <path to psd1>`


- Run the cmd: `Get-Command -Module microsoft.applicationinsights.iisconfigurator.poc` to get the available commands

	```
	CommandType     Name                                               Version    Source
	-----------     ----                                               -------    ------
	Cmdlet          Disable-ApplicationInsightsMonitoring              0.1.0      Microsoft.ApplicationInsights.IISConfigurator.POC
	Cmdlet          Enable-ApplicationInsightsMonitoring               0.1.0      Microsoft.ApplicationInsights.IISConfigurator.POC
	Cmdlet          Get-ApplicationInsightsMonitoringStatus            0.1.0      Microsoft.ApplicationInsights.IISConfigurator.POC
	```


- Run the cmd: `Get-ApplicationInsightsMonitoringStatus` to get an output of information about this module.

	```
	PowerShell Module version:
	0.1.0-alpha

	Application Insights SDK version:
	2.9.0.0

	Executing PowerShell Module Assembly:
	Microsoft.ApplicationInsights.Redfield.Configurator.PowerShell, Version=2.8.13.5662, Culture=neutral, PublicKeyToken=31bf3856ad364e35

	PowerShell Module Directory:
	C:\Program Files\WindowsPowerShell\Modules\Microsoft.ApplicationInsights.IISConfigurator.POC\content\PowerShell

	Runtime Paths:
	ParentDirectory: C:\Program Files\WindowsPowerShell\Modules\Microsoft.ApplicationInsights.IISConfigurator.POC\content Exists: True
	ConfigurationPath: C:\Program Files\WindowsPowerShell\Modules\Microsoft.ApplicationInsights.IISConfigurator.POC\content\applicationInsights.ikey.config Exists: False
	ManagedHttpModuleHelperPath: C:\Program Files\WindowsPowerShell\Modules\Microsoft.ApplicationInsights.IISConfigurator.POC\content\Runtime\Microsoft.AppInsights.IIS.ManagedHttpModuleHelper.dll Exists: True
	RedfieldIISModulePath: C:\Program Files\WindowsPowerShell\Modules\Microsoft.ApplicationInsights.IISConfigurator.POC\content\Runtime\Microsoft.ApplicationInsights.RedfieldIISModule.dll Exists: True
	InstrumentationEngine86Path: C:\Program Files\WindowsPowerShell\Modules\Microsoft.ApplicationInsights.IISConfigurator.POC\content\Instrumentation32\MicrosoftInstrumentationEngine_x86.dll Exists: True
	InstrumentationEngine64Path: C:\Program Files\WindowsPowerShell\Modules\Microsoft.ApplicationInsights.IISConfigurator.POC\content\Instrumentation64\MicrosoftInstrumentationEngine_x64.dll Exists: True
	InstrumentationEngineExtensionHost86Path: C:\Program Files\WindowsPowerShell\Modules\Microsoft.ApplicationInsights.IISConfigurator.POC\content\Instrumentation32\Microsoft.ApplicationInsights.ExtensionsHost_x86.dll Exists: True
	InstrumentationEngineExtensionHost64Path: C:\Program Files\WindowsPowerShell\Modules\Microsoft.ApplicationInsights.IISConfigurator.POC\content\Instrumentation64\Microsoft.ApplicationInsights.ExtensionsHost_x64.dll Exists: True
	InstrumentationEngineExtensionConfig86Path: C:\Program Files\WindowsPowerShell\Modules\Microsoft.ApplicationInsights.IISConfigurator.POC\content\Instrumentation32\Microsoft.InstrumentationEngine.Extensions.config Exists: True
	InstrumentationEngineExtensionConfig64Path: C:\Program Files\WindowsPowerShell\Modules\Microsoft.ApplicationInsights.IISConfigurator.POC\content\Instrumentation64\Microsoft.InstrumentationEngine.Extensions.config Exists: True
	ApplicationInsightsSdkPath: C:\Program Files\WindowsPowerShell\Modules\Microsoft.ApplicationInsights.IISConfigurator.POC\content\Runtime\Microsoft.ApplicationInsights.dll Exists: True
	```




### Troubleshooting Running Processes

Can inspect the process on the instrumented machine to see if all DLLs are loaded.
If attach is working, 17 DLLS should be loaded.

- Cmd: `Get-ApplicationInsightsMonitoringStatus -InspectProcess`

	```
	iisreset.exe /status
	Status for IIS Admin Service ( IISADMIN ) : Running
	Status for Windows Process Activation Service ( WAS ) : Running
	Status for Net.Msmq Listener Adapter ( NetMsmqActivator ) : Running
	Status for Net.Pipe Listener Adapter ( NetPipeActivator ) : Running
	Status for Net.Tcp Listener Adapter ( NetTcpActivator ) : Running
	Status for World Wide Web Publishing Service ( W3SVC ) : Running

	handle64.exe -accepteula -p w3wp
	  BF0: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\microsoft.applicationinsights.iisconfigurator.poc\content\Runtime\Microsoft.AI.ServerTelemetryChannel.dll
	  C58: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\microsoft.applicationinsights.iisconfigurator.poc\content\Runtime\Microsoft.AI.AzureAppServices.dll
	  C68: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\microsoft.applicationinsights.iisconfigurator.poc\content\Runtime\Microsoft.AI.DependencyCollector.dll
	  C78: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\microsoft.applicationinsights.iisconfigurator.poc\content\Runtime\Microsoft.AI.WindowsServer.dll
	  C98: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\microsoft.applicationinsights.iisconfigurator.poc\content\Runtime\Microsoft.AI.Web.dll
	  CBC: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\microsoft.applicationinsights.iisconfigurator.poc\content\Runtime\Microsoft.AI.PerfCounterCollector.dll
	  DB0: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\microsoft.applicationinsights.iisconfigurator.poc\content\Runtime\Microsoft.AI.Agent.Intercept.dll
	  B98: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\microsoft.applicationinsights.iisconfigurator.poc\content\Runtime\Microsoft.ApplicationInsights.RedfieldIISModule.dll
	  BB4: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\microsoft.applicationinsights.iisconfigurator.poc\content\Runtime\Microsoft.ApplicationInsights.RedfieldIISModule.Contracts.dll
	  BCC: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\microsoft.applicationinsights.iisconfigurator.poc\content\Runtime\Microsoft.ApplicationInsights.Redfield.Lightup.dll
	  BE0: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\microsoft.applicationinsights.iisconfigurator.poc\content\Runtime\Microsoft.ApplicationInsights.dll

	listdlls64.exe -accepteula w3wp
	0x0000000019ac0000  0x127000  C:\Program Files\WindowsPowerShell\Modules\microsoft.applicationinsights.iisconfigurator.poc\content\Instrumentation64\MicrosoftInstrumentationEngine_x64.dll
	0x00000000198b0000  0x4f000   C:\Program Files\WindowsPowerShell\Modules\microsoft.applicationinsights.iisconfigurator.poc\content\Instrumentation64\Microsoft.ApplicationInsights.ExtensionsHost_x64.dll
	0x000000000c460000  0xb2000   C:\Program Files\WindowsPowerShell\Modules\microsoft.applicationinsights.iisconfigurator.poc\content\Instrumentation64\Microsoft.ApplicationInsights.Extensions.Base_x64.dll
	0x000000000ad60000  0x108000  C:\Windows\TEMP\2.4.0.0.Microsoft.ApplicationInsights.Extensions.Intercept_x64.dll
	```

### Collect ETW Logs with PerfView

#### Setup

- Download PerfView.exe and PerfView64.exe from https://github.com/Microsoft/perfview/releases
- Launch PerfView64.exe
- Expand "Advanced Options"
- Uncheck:
	- Zip
	- Merge
	- .NET Symbol Collection
- Set Additional Providers: 61f6ca3b-4b5f-5602-fa60-759a2a2d1fbd,323adc25-e39b-5c87-8658-2c1af1a92dc5,925fa42b-9ef6-5fa7-10b8-56449d7a2040,f7d60e07-e910-5aca-bdd2-9de45b46c560,7c739bb9-7861-412e-ba50-bf30d95eae36,61f6ca3b-4b5f-5602-fa60-759a2a2d1fbd,323adc25-e39b-5c87-8658-2c1af1a92dc5,252e28f4-43f9-5771-197a-e8c7e750a984


#### Collecting Logs

- in a cmd window with admin priveleges , execute `iisreset /stop` To turn off IIS and all web apps.
- In PerfView, click "Start Collection"
- in a cmd window with admin priveleges , execute `iisreset /start` To start IIS.
- try to browse to your app.
- after your app is loaded, In PerfView, click "Stop Collection"

