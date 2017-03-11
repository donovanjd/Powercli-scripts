Connect-VIServer -Server 192.168.10.1 -User root -Password supersecret
$cluster = Read-Host "Enter cluster name:"
$esxhosts = Get-Cluster $cluster | Get-VMHost
$esxcreds = Get-Credential
$datastore = "iscsi-storage"

foreach ($esx in $esxhosts){
Write-host "Connecting to:" $esx
$ssh = Posh-SSH\New-SSHSession -ComputerName $esx -Credential $esxcreds -AcceptKey:$true
Write-host "SSH connection established to:" $esx -ForegroundColor Green
Write-host "Stopping vShield-Stateful-Firewall service" -ForegroundColor Green
Invoke-SSHCommand -SSHSession $ssh -Command (" /etc/init.d/vShield-Stateful-Firewall stop")

Write-host "Backup original vShield-Stateful-Firewall config file" -ForegroundColor Green
Invoke-SSHCommand -SSHSession $ssh -Command (" cp /etc/init.d/vShield-Stateful-Firewall /etc/init.d/vShield-Stateful-Firewall.orig")

Write-host "Copying file from the datastore to /tmp" -ForegroundColor Green
Invoke-SSHCommand -SSHSession $ssh -Command (" cp /vmfs/volumes/$datastore/vShield-Stateful-Firewall /tmp/")

Write-host "Starting vShield-Stateful-Firewall service from the /tmp folder" -ForegroundColor Green
Invoke-SSHCommand -SSHSession $ssh -Command (" /tmp/vShield-Stateful-Firewall start")
Start-Sleep -s 10

Write-host "Check if vsfwd is running" -ForegroundColor Green
$service_stat = Invoke-SSHCommand -SSHSession $ssh -Command (" /tmp/vShield-Stateful-Firewall status")
write-host $service_stat.output

Write-host "Editing local.sh to make this change persistent on reboot." -ForegroundColor Green
Invoke-SSHCommand -SSHSession $ssh -Command ("sed -i '/exit/i cp /vmfs/volumes/$datastore/vShield-Stateful-Firewall /tmp/' /etc/rc.local.d/local.sh”)
Invoke-SSHCommand -SSHSession $ssh -Command ("sed -i '/exit/i /tmp/vShield-Stateful-Status restart’  /etc/rc.local.d/local.sh”)

Write-host "Validating vsfwd memory config" -ForegroundColor Green
$return = Invoke-SSHCommand -SSHSession $ssh -Command ("vsish -e set /sched/groupPathNameToID host vim vmvisor vsfwd")
$groupid = $return.output[0].ToString().Trim()
Write-Host "vsfwd group id:" $groupid

$return1 = Invoke-SSHCommand -SSHSession $ssh -Command ("vsish -e get /sched/groups/$groupid/memAllocationInMB")
$return1.output
}
Remove-SSHSession -SSHSession $ssh
