Get-PingStatus

I have two advanced functions 'ValueFromPipeline' and 'ValidateSet'.
'ValueFromPipeline' gives the capability to pass objects to our script. Allowing me to pass many devices, in my case, to the 'ping' function.
 Other than the message "Online: PC1", I wanted to be able to use the status result to pass to another cmdlet, collate all online or offline devices and display the results in a table.
 Using 'ValidateSet' I could define my options, "Online","Offline" and "ObjectTable". But it’s not set to mandatory, so I don't have use it.
To use device status objects, I needed to hold them somewhere, so I created an empty array at the start. Regardless of what option I choose, if any, the below block of code will always run:

$device| foreach {
            if (Test-Connection $_ -Count 1 -Quiet) {
                if(-not($GetObject)){write-host -ForegroundColor green "Online: $_ "}
                    $Hash = $Hash += @{Online="$_"}
            }else{
                if(-not($GetObject)){write-host -ForegroundColor Red "Offline: $_ "}
                    $Hash = $Hash += @{Offline="$_"}
                }
        }
        
Whatever is held in the variable $device, each one will 'pinged' pass through a 'if' statement then go to either offline or online and get stored into the $hash array variable.
DISCLAIMER: I should apologies to Don here for killing the puppies with write-host. I wanted to just push out some coloured output to the host only!

Before I go any further, let me briefly explain how I am 'pinging' the devices. I am using the cmdlet 'Test-Connection'. The synopsis on 'get-help' for test-connection states, 'Sends ICMP echo request packets ("pings") to one or more computers.' A nice feature of this cmdlet is '-quiet' syntax. This is cool as it gives a Boolean result (True or False) of the 'ping' status. By adding a '-count' as well i can limit the number of times I request a connection check. 
Now I can pass as many devices through the pipeline to my function and get an online or offline message pretty quickly.
The reminder of the script uses an if statement on the '$getObject' parameter. the use of the 'validateSet' allows me to make sure the three options are used only. 

The data collected in the $hash variable is passed through a foreach statement and creates customobjects. The final part is use of a 'Switch'. Depending on what was chosen in the $getObject parameter will displayed at the end of the script.
The advantage to this switch is I can pass all the online PC's to something else via the pipeline. For example, an AD group or a deployment collection:
'PC1','PC2' | Get-PingStatus -GetObject Online | # pass to another cmdlet 

Capture the 'online' PC's to a variable:
$Online = 'PC1','PC2' | Get-PingStatus -GetObject Online

Or if you need to report back a list of PC's status's in an object group:
'PC1','PC2' | Get-PingStatus -GetObject objectTable
DeviceName Online offline
---------- ------ -------
pc4        Online
pc1               Offline
pc2               Offline
pc3               Offline

Again this script has great flexibility in how you pass the device objects.
Say you have a list of PC's in a text for CSV file, use Get-content and pipe it to Get-PingStatus:
get-content pcs.csv | Get-PingStatus
I hope you've enjoyed my first blog and i welcome any comments. I've posted the script on GitHub for download.


