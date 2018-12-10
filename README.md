# Listen to ATD Reports that get broadcast on OpenDXL

McAfee Advanced Threat Defense can be configured to broadcast everytime it identifies malware, or completes a sample. This is similiar to a registered webhook, but better because it is on the DXL (Pub/Sub framework). 

```/mcafee/event/atd/file/report```

You need to install the EPO Extension for `ATDDXLTags`. This will allow for management of the channel.

![install](epo_extension/extension-installed.png?raw=true)

This extension is listed as supported under EPO 5.9.x [KB87142](https://kc.mcafee.com/corporate/index?page=content&id=KB87142&actp=null&viewlocale=en_US)

Products → Downloads → My Products → All Versions → ATD 4.4

```
SHA256(MATDDXLTag.zip)= 556ed2fee310f1e6e80c04b085954e0dd7bb2ae1e1bb977c542bb26b2a86ed9b
```

# General McAfee Information

* https://support.mcafee.com/ServicePortal/faces/knowledgecenter?p=Advanced+Threat+Defense

* https://kc.mcafee.com/resources/sites/MCAFEE/content/live/PRODUCT_DOCUMENTATION/27000/PD27380/en_US/MATD_420_CLI_Reference_Guide_en-us.pdf

Try using the CLI command `dxlstatus`, `clearstats all`, &  `restart dxlservice` on the ATD to troubleshoot the DXL.

```
> dxlstatus
<=== DXL STATUS ===>
Status : DISABLED
DXL Channel Status : DOWN
Sample Files Received Count : 0
Sample Files Published Count : 0
Sample Files Queued Count : 0

> clearstats all
 
 <=== DXL STATUS ===>
Status : DISABLED
DXL Channel Status : DOWN
Sample Files Received Count : 0
Sample Files Published Count : 0
Sample Files Queued Count : 0
 ```

Additionally you will want to ensure this is correct

 `show tepublisherstatus`

```
********ePO Threat Event Publisher Status********
 tepublisher is not running
```
# Configure Requirements

## EPO DXL Permissions

DXL Permissions are implicit and the `All Systems` permission works even with certificates, not just tag or ma present machines. If you limit the read only permissions of the report, you must explicitly at least allow the TIE server to read the channel, or create a common tag for all systems.

```/mcafee/event/atd/file/report```

![permissions](epo_extension/extension.png?raw=true)

## EPO System Tree

![systemtree](epo_extension/extension-systemtree.png?raw=true)

## McAfee ATD

![atd](epo_extension/ATD_TIE_Publish.png?raw=true)

# Install

Install `Virtualenv`  (Ubuntu with python27)

```
shadowbq@space-junk:~$ sudo apt install python-pip
shadowbq@space-junk:~$ pip -V
shadowbq@space-junk:~$ pip install --upgrade pip 
shadowbq@space-junk:~$ pip install virtualenv
```

Create and Activate the `Virtualenv`

```
shadowbq@space-junk:~$ virtualenv tietroubleshoot
New python executable in /home/shadowbq/tietroubleshoot/bin/python
Installing setuptools, pip, wheel...done.
shadowbq@space-junk:~$ source tietroubleshoot/bin/activate
(tietroubleshoot) shadowbq@space-junk:~$ 
(tietroubleshoot) shadowbq@space-junk:~$ git clone https://github.com/shadowbq/opendxl-atd-troubleshoot 
```

Install the python 'requirments.txt' file

```
:~/opendxl-atd-troubleshoot$ pip install -r requirements.txt 
Collecting dxlclient (from -r requirements.txt (line 1))
  Downloading dxlclient-4.0.0.416-py2.7-none-any.whl (121kB)
    100% |████████████████████████████████| 122kB 4.1MB/s 
Collecting requests (from dxlclient->-r requirements.txt (line 1))
  Using cached requests-2.18.4-py2.py3-none-any.whl
Collecting six (from dxlclient->-r requirements.txt (line 1))
  Using cached six-1.11.0-py2.py3-none-any.whl
Collecting asn1crypto (from dxlclient->-r requirements.txt (line 1))
  Downloading asn1crypto-0.23.0-py2.py3-none-any.whl (99kB)
    100% |████████████████████████████████| 102kB 5.6MB/s 
Collecting oscrypto (from dxlclient->-r requirements.txt (line 1))
  Downloading oscrypto-0.18.0-py2.py3-none-any.whl (175kB)
    100% |████████████████████████████████| 184kB 4.6MB/s 
Collecting configobj (from dxlclient->-r requirements.txt (line 1))
  Downloading configobj-5.0.6.tar.gz
Collecting urllib3<1.23,>=1.21.1 (from requests->dxlclient->-r requirements.txt (line 1))
  Using cached urllib3-1.22-py2.py3-none-any.whl
Collecting idna<2.7,>=2.5 (from requests->dxlclient->-r requirements.txt (line 1))
  Using cached idna-2.6-py2.py3-none-any.whl
Collecting chardet<3.1.0,>=3.0.2 (from requests->dxlclient->-r requirements.txt (line 1))
  Using cached chardet-3.0.4-py2.py3-none-any.whl
Collecting certifi>=2017.4.17 (from requests->dxlclient->-r requirements.txt (line 1))
  Using cached certifi-2017.11.5-py2.py3-none-any.whl
Building wheels for collected packages: configobj
  Running setup.py bdist_wheel for configobj ... done
  Stored in directory: /home/shadowbq/.cache/pip/wheels/87/76/48/1564f8466fbd36402af5ac4972ffb56a6ef7f143892ef57fe5
Successfully built configobj
Installing collected packages: urllib3, idna, chardet, certifi, requests, six, asn1crypto, oscrypto, configobj, dxlclient
Successfully installed asn1crypto-0.23.0 certifi-2017.11.5 chardet-3.0.4 configobj-5.0.6 dxlclient-4.0.0.416 idna-2.6 oscrypto-0.18.0 requests-2.18.4 six-1.11.0 urllib3-1.22
```

# Configure

Fix the locations for the DXLclient config 

```
~/opendxl-atd-troubleshoot/config$ cat dxlcient.config 
[Certs]
BrokerCertChain=$HOME/opendxl-atd-troubleshoot/config/broker.crt

CertFile=$HOME/opendxl-atd-troubleshoot/config/client.crt
PrivateKey=$HOME/opendxl-atd-troubleshoot/config/client.key

[Brokers]
{d08fb638-208e-11e7-1a34-005056a4473a}={d08fb638-208e-11e7-1a34-005056a4473a};8883;poddxlbroker1;10.1.0.11
{0060f20a-208a-11e7-3adf-005056a4270a}={0060f20a-208a-11e7-3adf-005056a4270a};8883;poddxlbroker2;10.1.0.12
```


# Example 

```
(tietroubleshoot) shadowbq@space-junk:~/opendxl-atd-troubleshoot$ python atd_subscriber.py 
client conntecting
client conntected
Subscribing: /mcafee/event/atd/file/report
Event received:
payload: {
"Summary": {
 	"Event_Type": "ATD File Report",
	"MISversion": "4.2.0.20",
	"SUMversion": "4.2.0.20",
	"DETversion": "4.2.0.171126",
	"OSversion": "win7sp1x64_newWin7",
	"fileId": "Not Available",
	"Parent MD5": "Not Available",
	"ATD IP": "10.0.0.252",
	"Src IP": "",
	"Dst IP": "",
	"TaskId": "73850",
	"JobId": "47594",
	"JSONversion": "1.002",
	"hasDynamicAnalysis": "true",
	"Subject": 	{
		"Name": "putty.exe.h17",
		"Type": "PE32 executable (GUI) Intel 80386",
		"FileType": "0",
		"md5": "DE62187CAD4158C49ECAD48F39D03561",
		"sha-1": "65CA95DED1E132873476D719009768E8E4F51F9A",
		"sha-256": "369992EB1DAA23644F96E81D381B52CB0C73E7F676262B730C3380BDD9B0A940",
		"size": "774200",
		"Timestamp": "2017-12-05 17:07:41",
		"parent_archive": "Not Available"
	},
	"Selectors": [
		{
			"Engine": "Gateway Anti-Malware",
			"MalwareName": "---",
			"Severity": "0"
		},
		{
			"Engine": "GTI File Reputation",
			"MalwareName": "---",
			"Severity": "0"
		},
		{
			"Engine": "Anti-Malware",
			"MalwareName": "---",
			"Severity": "0"
		},
		{
			"Engine": "YARA",
			"MalwareName": "HappyBunnyRule",
			"Severity": "5"
		},
		{
			"Engine": "CustomRules",
			"MalwareName": "Malware.Dynamic",
			"Severity": "4"
		},
		{
			"Engine": "Sandbox",
			"MalwareName": "Malware.Dynamic",
			"Severity": "4"
		}
	],
	"Verdict": {
		"Severity": "5",
		"Description": "Sample is malicious: final severity level 5"
	},
	"Processes" : [
		{
			"Name" : "putty.exe.h17",
			"Md5" : "DE62187CAD4158C49ECAD48F39D03561",
			"Sha1" : "65CA95DED1E132873476D719009768E8E4F51F9A",
			"Sha256" : "369992EB1DAA23644F96E81D381B52CB0C73E7F676262B730C3380BDD9B0A940",
			"Severity" : "4"
		}
	],
	"Stats": [
		{
			"ID": "0",
			"Category": "Persistence, Installation Boot Survival ",
			"Severity": "3"
		},
		{
			"ID": "1",
			"Category": "Hiding, Camouflage, Stealthiness, Detection and Removal Protection ",
			"Severity": "1"
		},
		{
			"ID": "2",
			"Category": "Security Solution / Mechanism bypass, termination and removal, Anti Debugging, VM Detection ",
			"Severity": "1"
		},
		{
			"ID": "3",
			"Category": "Spreading ",
			"Severity": "0"
		},
		{
			"ID": "4",
			"Category": "Exploiting, Shellcode ",
			"Severity": "4"
		},
		{
			"ID": "5",
			"Category": "Networking ",
			"Severity": "3"
		},
		{
			"ID": "6",
			"Category": "Data spying, Sniffing, Keylogging, Ebanking Fraud ",
			"Severity": "4"
		}
	],
	"Behavior": [
		{
			"ID": "2",
			"Analysis": "Identified as --- by Gateway Anti-Malware"
		},
		{
			"ID": "1",
			"Analysis": "Identified as --- by GTI File Reputation"
		},
		{
			"ID": "4",
			"Analysis": "Identified as --- by Anti-Malware"
		},
		{
			"ID": "128",
			"Analysis": "Identified as HappyBunnyRule by YARA"
		},
		{
			"ID": "5001",
			"Analysis": " SimTa Memory Scrape  [MalMemoryCollect]"
		},
		{
			"ID": "5001",
			"Analysis": " Find PuTTY via reg read access  [PuttyRegKeyAccess]"
		},
		{
			"ID": "5001",
			"Analysis": " SimTa Shellc0de detected  [SimonShellCode]"
		},
		{
			"ID": "17",
			"Analysis": "Sample signed with invalid certificate"
		},
		{
			"ID": "262",
			"Analysis": "Obtained user's logon name"
		},
		{
			"ID": "265",
			"Analysis": "Disabled attach/detach notifications from dynamic link library"
		}
	]
	}
}

```
# Changing TIE Server Log Levels

It may also help to change the current log level of the TIE server, to also see additional information.

You can find information about the files that are sent to ATD on the TIE server at `/var/McAfee/tieserver/logs/tieserver.log`.


## TIE Server 2.x

Change the log level in the *UI of EPO* with a Policy Setting for the TIE Server.

On the Server Configuration tab, configure the logging level of the server, enable or disable collecting
metrics, and modify the sampling period for collecting performance metrics.

You may want to restart the TIE server or service

## TIE Server 1.x

To increase the verbosity of the ATD logging:
Use a text editor to uncomment the following lines in `/opt/McAfee/tieserver/conf/log4j.properties`:

```
#log4j.logger.com.mcafee.jti.server.service.atd=trace, file
#log4j.additivity.com.mcafee.jti.server.service.atd=false

log4j.logger.com.mcafee.jti.server.service.atd=trace, file
log4j.additivity.com.mcafee.jti.server.service.atd=false
 ```
 
Use the following command to restart the TIE server service:

```
$> service tieserver restart
```



