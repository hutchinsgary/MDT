# MDT
Microsoft Deployment Toolkit





Code

 ###### Form Functions ######



function Get-MDTComputer


{


	[CmdletBinding()]


	PARAM


	(


		[Parameter(ValueFromPipelineByPropertyName = $true)]


		$id = "",


		[Parameter(ValueFromPipelineByPropertyName = $true)]


		$assetTag = "",


		[Parameter(ValueFromPipelineByPropertyName = $true)]


		$macAddress = "",


		[Parameter(ValueFromPipelineByPropertyName = $true)]


		$serialNumber = "",


		[Parameter(ValueFromPipelineByPropertyName = $true)]


		$uuid = "",


		[Parameter(ValueFromPipelineByPropertyName = $true)]


		$description = ""


	)


	


	Process


	{


		# Build a select statement based on what parameters were specified


		if ($id -eq "" -and $assetTag -eq "" -and $macAddress -eq "" -and $serialNumber -eq "" -and $uuid -eq "" -and $description -eq "")


		{


			$sql = "SELECT * FROM ComputerSettings"


		}


		elseif ($id -ne "")


		{


			$sql = "SELECT * FROM ComputerSettings WHERE ID = $id"


		}


		else


		{


			# Specified the initial command


			$sql = "SELECT * FROM ComputerSettings WHERE "


			


			# Add the appropriate where clauses


			if ($assetTag -ne "")


			{


				$sql = "$sql AssetTag='$assetTag' AND"


			}


			


			if ($macAddress -ne "")


			{


				$sql = "$sql MacAddress='$macAddress' AND"


			}


			


			if ($serialNumber -ne "")


			{


				$sql = "$sql SerialNumber='$serialNumber' AND"


			}


			


			if ($uuid -ne "")


			{


				$sql = "$sql UUID='$uuid' AND"


			}


			


			if ($description -ne "")


			{


				$sql = "$sql Description='$description' AND"


			}


			


			# Chop off the last " AND"


			$sql = $sql.Substring(0, $sql.Length - 4)


		}


		


		$selectAdapter = New-Object System.Data.SqlClient.SqlDataAdapter($sql, $mdDataCenterLConnection)


		$selectDataset = New-Object System.Data.Dataset


		$null = $selectAdapter.Fill($selectDataset, "ComputerSettings")


		$selectDataset.Tables[0].Rows


	}


}



function New-MDTComputer


{


	


	[CmdletBinding()]


	PARAM


	(


		[Parameter(ValueFromPipelineByPropertyName = $true)]


		$assetTag,


		[Parameter(ValueFromPipelineByPropertyName = $true)]


		$macAddress,


		[Parameter(ValueFromPipelineByPropertyName = $true)]


		$serialNumber,


		[Parameter(ValueFromPipelineByPropertyName = $true)]


		$uuid,


		[Parameter(ValueFromPipelineByPropertyName = $true)]


		$description,


		[Parameter(ValueFromPipelineByPropertyName = $true, Mandatory = $true)]


		$settings


	)


	


	Process


	{


		# Insert a new computer row and get the identity result


		$sql = "INSERT INTO ComputerIdentity (AssetTag, SerialNumber, MacAddress, UUID, Description) VALUES ('$assetTag', '$serialNumber', '$macAddress', '$uuid', '$description') SELECT @@IDENTITY"


		Write-Verbose "About to execute command: $sql"


		$identityCmd = New-Object System.Data.SqlClient.SqlCommand($sql, $mdDataCenterLConnection)


		$identity = $identityCmd.ExecuteScalar()


		Write-Verbose "Added computer identity record"


		


		# Insert the settings row, adding the values as specified in the hash table


		$settingsColumns = $settings.Keys -join ","


		$settingsValues = $settings.Values -join "','"


		$sql = "INSERT INTO Settings (Type, ID, $settingsColumns) VALUES ('C', $identity, '$settingsValues')"


		Write-Verbose "About to execute command: $sql"


		$settingsCmd = New-Object System.Data.SqlClient.SqlCommand($sql, $mdDataCenterLConnection)


		$null = $settingsCmd.ExecuteScalar()


		


		# Write the new record back to the pipeline


		Get-MDTComputer -ID $identity


	}


}



function Remove-MDTComputer


{	


	[CmdletBinding()]


	PARAM


	(


		[Parameter(ValueFromPipelineByPropertyName = $true, Mandatory = $true)]


		$id


	)


	


	Process


	{


		# Build the delete command


		$delCommand = "DELETE FROM ComputerIdentity WHERE ID = $id"


		


		# Issue the delete command


		Write-Verbose "About to issue command: $delCommand"


		$cmd = New-Object System.Data.SqlClient.SqlCommand($delCommand, $mdDataCenterLConnection)


		$null = $cmd.ExecuteScalar()


	}


}



function Validate-IPDetails


{


	if ((Test-IsInSameSubnet -ip1 $IP_textbox.Text -ip2 $Gateway_textbox.Text -mask $Subnet_textbox.Text) -eq $true)


	{


		$IP_textbox.BackColor = 'PaleGreen'


		$Gateway_textbox.BackColor = 'PaleGreen'


		$Subnet_textbox.BackColor = 'PaleGreen'


	}


	else


	{


		$IP_textbox.BackColor = 'LightSalmon'


		$Gateway_textbox.BackColor = 'LightSalmon'


		$Subnet_textbox.BackColor = 'LightSalmon'


	}


	if ($IP_textbox.Text -eq $Gateway_textbox.Text)


	{


		$IP_textbox.BackColor = 'LightSalmon'


		$Gateway_textbox.BackColor = 'LightSalmon'


	}


}



　


function Get-NetwotkAddress


{


	param (


		[IpAddress]$ip,


		[IpAddress]$Mask


	)


	


	$IpAddressBytes = $ip.GetAddressBytes()


	$SubnetMaskBytes = $Mask.GetAddressBytes()


	


	if ($IpAddressBytes.Length -ne $SubnetMaskBytes.Length)


	{


		throw "Lengths of IP address and subnet mask do not match."


		exit 0


	}


	


	$BroadcastAddress = @()


	


	for ($i = 0; $i -le 3; $i++)


	{


		$BroadcastAddress += $ipAddressBytes[$i] -band $subnetMaskBytes[$i]


		


	}


	


	$BroadcastAddressString = $BroadcastAddress -Join "."


	return [IpAddress]$BroadcastAddressString


}



function Test-IsInSameSubnet


{


	param (


		[IpAddress]$ip1,


		[IpAddress]$ip2,


		[IpAddress]$mask


	)


	


	$Network1 = Get-NetwotkAddress -ip $ip1 -mask $mask


	$Network2 = Get-NetwotkAddress -ip $ip2 -mask $mask


	


	return $Network1.Equals($Network2)


}



function Test-IP


{


	param (


		[IpAddress]$ip1


	)


	


	if (Test-Connection -ComputerName $ip1 -Quiet)


	{


		$IP_textbox.BackColor = 'LightSalmon' }


	


	Else


	{


		$IP_textbox.BackColor = 'PaleGreen' }


}



　


function Validate-Form


{


	[int]$failcount = 0


	foreach ($control in $SGBM_Build_FrontEnd.Controls)


	{


		foreach ($childcontrol in $control)


		{


			if ($childcontrol.BackColor -match 'LightSalmon')


			{


				$failcount++


			}


		}


	}


	if ($failcount -eq 0)


	{


		return $true


		#Validation Passed


	}


	else


	{


		return $false


	}


}



　


Function Get-Computer ($Computername, $Domain, $Delete)


{


	$gcbdomain = [System.DirectoryServices.DirectoryEntry]'LDAP://DC=domain,DC=domain,DC=domain,DC=domain'


	$search = [System.DirectoryServices.DirectorySearcher]$Domain


	$search.searchroot = $gcbdomain


	$search.filter = "(&(objectclass=computer)(name=$Computername))"


	


	If ($delete)


	{


		$search.findall() | %{ $_.GetDirectoryEntry() } | %{ $_.DeleteObject(0) }


		$Result = $search.findall() | %{ $_.GetDirectoryEntry() }


	}


	Else


	{


		$Result = $search.findall() | %{ $_.GetDirectoryEntry() }


	}


	


	


	If ($Result)


	{


		$True


		$Result.distinguishedName


	}


	Else


	{


		Return $False


	}


}



　


function Get-ScriptDirectory


{


	if ($hostinvocation -ne $null)


	{


		Split-Path $hostinvocation.MyCommand.path


	}


	else


	{


		Split-Path $script:MyInvocation.MyCommand.Path


	}


}



###### END Form Functions ######



　


　


$SGBM_Build_FrontEnd_Load={


	


	# Mainform Load


	# Get info from the FrontEnd Database Table and populate the dropdowns


	


	Clear-Variable -name mdtDatabase -errorAction SilentlyContinue


	


	$SQLServer = "sqlserver,1504"


	$SQLDBName = "MDT_PRO"


	$SqlQuery = "select * from FrontEnd"


	


	$global:mdDataCenterLConnection = New-Object System.Data.SqlClient.SqlConnection


	$mdDataCenterLConnection.ConnectionString = "Server = $SQLServer; Database = $SQLDBName; Integrated Security = True"


	$mdDataCenterLConnection.Open()


	


	$SqlCmd = New-Object System.Data.SqlClient.SqlCommand


	$SqlCmd.CommandText = $SqlQuery


	$SqlCmd.Connection = $global:mdDataCenterLConnection


	


	$SqlAdapter = New-Object System.Data.SqlClient.SqlDataAdapter


	$SqlAdapter.SelectCommand = $SqlCmd


	


	$DataSet = New-Object System.Data.DataSet


	$SqlAdapter.Fill($DataSet)


	


	#$SqlConnection.Close()


	


	$OS = $DataSet.Tables[0] | select -ExpandProperty OS


	$DataCenter = $DataSet.Tables[0] | select -ExpandProperty TaskSequenceID


	$ENV = $DataSet.Tables[0] | select -ExpandProperty Environment


	$APP = $DataSet.Tables[0] | select -ExpandProperty AppProfile


	$DOM = $DataSet.Tables[0] | select -ExpandProperty Domain


	$LOC = $DataSet.Tables[0] | select -ExpandProperty Location


	$DNS = $DataSet.Tables[0] | select -ExpandProperty DNSServers


	$DNSSUF = $DataSet.Tables[0] | select -ExpandProperty DNSSuffix


	$NTPSRV = $DataSet.Tables[0] | select -ExpandProperty NTPServers


	


	


# Populate the Dropdowns on the form


	


	# Populate OS Dropdown


	$objOSArray = @() #define object array


	$I = 0


	


	foreach ($O in $OS)


	{	If ($O -ne "")


		{	


			$obj = new-object System.Object


			$obj | add-member -type NoteProperty -name OS -value $OS[$I]


			$obj | add-member -type NoteProperty -name DataCenter -value $DataCenter[$I]


			$I++


			$objOSArray += $obj


		}


	}


	


	$OS_combobox.items.addrange($objOSArray)


	$OS_combobox.ValueMember = "DataCenter"


	$OS_combobox.DisplayMember = "OS"


	


	$obj =  ""


	# END Populate OS Dropdown


	


	# Populate Environment Dropdown (Linked with the DNS Suffix)


	$objENVArray = @()


	$I = 0


	


	foreach ($E in $ENV)


	{	If ($E -ne "")


		{


			$obj = new-object System.Object


			$obj | add-member -type NoteProperty -name ENV -value $ENV[$I]


			$obj | add-member -type NoteProperty -name DNSSUF -value $DNSSUF[$I]


			$I++


			$objENVArray += $obj


		}


		


	}


	


	$Environment_combobox.items.addrange($objENVArray)


	$Environment_combobox.ValueMember = "DNSSUF"


	$Environment_combobox.DisplayMember = "ENV"


	$obj = ""


	# END Populate Environment Dropdown


	


	# Populate AppProfile Dropdown


	$objAPPArray = @()


	$I = 0


	


	foreach ($A in $APP)


	{	If ($A -ne "")


		{


			$obj = new-object System.Object


			$obj | add-member -type NoteProperty -name APP -value $APP[$I]


			$I++


			$objAPPArray += $obj


		}


	}


	


	$APPProfile_combobox.items.addrange($objAPPArray)


	$APPProfile_combobox.DisplayMember = "APP"


	$obj = ""


	# END Populate AppProfile Dropdown


	


	# Populate Domain Dropdown


	$objDOMArray = @()


	$I = 0


	


	foreach ($D in $DOM)


	{


		If ($D -ne "")


		{


			$obj = new-object System.Object


			$obj | add-member -type NoteProperty -name DOM -value $DOM[$I]


			$I++


			$objDOMArray += $obj


		}


	}


	


	$Domain_combobox.items.addrange($objDOMArray)


	$Domain_combobox.DisplayMember = "DOM"


	$obj = ""


	


	# Set the default Value


	$Domain_combobox.SelectedIndex = 0


	


	# END Populate Domain Dropdown	


	


	# Populate Disk Unit Dropdown


	$DiskUnit_combobox.Items.Add("%")


	$DiskUnit_combobox.Items.Add("GB")


	


		# Set the Disk initial Values


	$DiskUnit_combobox.SelectedIndex = 0


	$DiskSize_textbox.Text = "100"


	$WipeDisks_checkbox.Checked = $true


	


	# END Populate Disk Unit Dropdown


	


	


	# Populate Location Dropdown (Linked with DNS / NTP Server value)


	$objLOCArray = @()


	$I = 0


	


	foreach ($L in $LOC)


	{


		If ($L -ne "")


		{


			$obj = new-object System.Object


			$obj | add-member -type NoteProperty -name LOC -value $LOC[$I]


			$obj | add-member -type NoteProperty -name DNSSERV -value $DNS[$I]


			$obj | add-member -type NoteProperty -name NTPSERV -value $NTPSRV[$I]


			$I++


			$objLOCArray += $obj


		}


		


	}


	


	$Location_combobox.items.addrange($objLOCArray)


	$Location_combobox.ValueMember = "DNSSERV"


	$Location_combobox.ValueMember = "NTPSERV"


	$Location_combobox.DisplayMember = "LOC"


	$obj = ""


	# END Populate Location Dropdown


}


# END Populate Dropdown's



　


　


# The following will run when the "Go" button is clicked


$Go_button_Click = {


	


	#Set Validation check counter to 0


	[int]$ValidationErr = 0


	


	# Declare in case we need a popup message


	$a = new-object -comobject wscript.shell


	


	# Clear Textbox


	$Output_richtextbox.Text = $Output_richtextbox.Text.Clear


	$Output_richtextbox.font = "lucida console"


	


	If (Validate-Form)


	{


		


		If ($WipeDisks_checkbox.Checked -eq $true)


		{


			$WipeDisk = "TRUE"


		}


		Else


		{


			$WipeDisk = "FALSE"


		}


		


		#Check additional application to be installed


		


		#.NET 3.5


		If ($NET35_checkbox.Checked -eq $true)


		{


			$App_NET35 = "TRUE"


		}


		Else


		{


			$App_NET35 = "FALSE"


		}


		


		#.NET 4.6.2


		If ($NET462_checkbox.Checked -eq $true)


		{


			$App_NET462 = "TRUE"


		}


		Else


		{


			$App_NET462 = "FALSE"


		}


		


		#Control-M


		If ($ControlM_checkbox.Checked -eq $true)


		{


			$App_CTM = "TRUE"


		}


		Else


		{


			$App_CTM = "FALSE"


		}


		


		#Netbackup


		If ($NetBackup_checkbox.Checked -eq $true)


		{


			$App_NETBCKP = "TRUE"


		}


		Else


		{


			$App_NETBCKP = "FALSE"


		}


		


		#Oracle 11 (x64)


		If ($Oracle11x64_checkbox.Checked -eq $true)


		{


			$App_ORA_11_x64 = "TRUE"


		}


		Else


		{


			$App_ORA_11_x64 = "FALSE"


		}


		


		If ($UEFI_checkbox.Checked -eq $true)


		{


			$UEFI = "TRUE"


		}


		Else


		{


			$UEFI = "FALSE"


		}


		


		#Oracle 11 (x86)


		If ($Oracle11X86_checkbox.Checked -eq $true)


		{


			$App_ORA_11_x86 = "TRUE"


		}


		Else


		{


			$App_ORA_11_x86 = "FALSE"


		}


		


		


		# Populate Variables based on form input


		$Hostname = $Hostname_textbox.Text


		$MACAddress = $MACAddress_textbox.Text


		$Domain = $Domain_combobox.SelectedItem.DOM


		$Environment = $Environment_combobox.SelectedItem.ENV


		$Description = $Description_textbox.Text


		$Operatingsystem = $OS_combobox.SelectedItem.OS


		$TaskSequence = $OS_combobox.SelectedItem.DataCenter


		$AppProfile = $APPProfile_combobox.SelectedItem.APP


		$IP = $IP_textbox.Text


		$Subnet = $Subnet_textbox.Text


		$Gateway = $Gateway_textbox.Text


		$DNSSrv = $DNSSrv_textbox.Text


		$DNSSuf = $DNSSuffix_textbox.Text


		$RequestNo = $RequestNo_textbox.Text


		[int]$DiskSize = $DiskSize_textbox.Text


		$DiskUnit = $DiskUnit_combobox.SelectedItem


		$NTPServers = $NTPServers_textbox.Text


		$AdminIP = $AdminIP_textbox.Text


			


		# Validation Checks


		


		# Check if GB has been specified as disk unit and convert to MB (this is because MDT does not create correct volume sizes in GB)


		If ($DiskUnit_combobox.Text -eq "GB")


		{


			$DiskUnit = "MB"


			[int]$DiskSize = $DiskSize * 1024


		}


		


		# Check if computer object exists in the AD


		


		$Q = Get-Computer -computer $Hostname -Domain $Domain


		If ($Q)


		{


			$DN = $Q[1]


			$intAnswer = $a.popup("Computer Account $Hostname exists in the Domain $Domain." + "`n`n$DN" + "`n`nDo you want to delete the existing record?", `


			0, "Delete AD Record", 4)


			If ($intAnswer -eq 6)


			{


				Try


				{


					Get-Computer -Computername $Hostname -Domain $Domain -Delete Y


				}


				Catch


				{


					$ValidationErr++  # Error Removing Record


				}


			}


			


			Else


			{	


					$ValidationErr++ # User specified NOT to delete the AD account (which could result in a failed build)


				}


			}		


	


		Else


		{


			# Do nothing as the AD record does not exist


		}


		$intAnswer = ""


		


		


		# Check if MDT Record exists already.  This will also check for duplicate entries in the DB


		


		$MDTComputers = Get-MDTComputer | Where-Object { $_.MacAddress -eq $MACAddress }


		


		If ($MDTComputers -ne $null)


		{


			$intAnswer = $a.popup("Servers exists in MDT.  Do you want to delete the existing record?", `


			0, "Delete MDT Record", 4)


			If ($intAnswer -eq 6)


			{


				foreach ($C in $MDTComputers)


				{


					$ID = $C.ID


					$Output_richtextbox.ForeColor = [Drawing.Color]::Red


					$Output_richtextbox.AppendText("`n")


					$Output_richtextbox.AppendText("`nExists in MDT with ID: $ID")


					$Output_richtextbox.AppendText("`nDeleting MDT record with ID: $ID")


					Remove-MDTComputer -id $ID


				}


			}


			


			$MDTComputers = Get-MDTComputer | Where-Object { $_.MacAddress -eq $MACAddress }


		}


		$intAnswer = ""


		


		# If validation checks are Ok then commit the server record to MDT


		If ($MDTComputers -eq $null -and $ValidationErr -eq 0)


		{


			#$Output_richtextbox.ForeColor = [Drawing.Color]::Green


			#$Output_richtextbox.AppendText("`n")


			#$Output_richtextbox.AppendText("`nNo record with $MACAddress")


			#$Output_richtextbox.AppendText("`n## Creating new record in MDT ##")


			


			#UEFI Check


			if ($UEFI -eq "FALSE")


			{


				$OSDPartitions = '1'


				$OSDPartitions0TYPE = 'Primary'


				$OSDPartitions0FILESYSTEM = 'NTFS'


				$OSDPartitions0BOOTABLE = 'TRUE'


				$OSDPartitions0QUICKFORMAT = 'TRUE'


				$OSDPartitions0VOLUMENAME = 'OSDisk'


				$OSDPartitions0VOLUMELETTERVARIABLE = 'OSDisk'


				$DoNotCreateExtraPartition = 'YES'


				$OSDPartitions0SIZE = $DiskSize;


				$OSDPartitions0SIZEUNITS = $DiskUnit;


			}


			elseif ($UEFI -eq "TRUE")


			{


			}


			


			


			New-MDTComputer -description $Hostname -macAddress $Macaddress.ToUpper() -settings @{


				OSInstall = 'YES';


				Engineer = $env:username;


				Environment = $Environment;


				Description = $Description;


				Build = $Operatingsystem;


				ComputerName = $Hostname;


				BaseBuildVersion = $Tasksequence;


				JoinDomain = $Domain;


				OSDComputerName = $Hostname;


				TaskSequenceID = $Tasksequence;


				ServiceRequest = $RequestNo;


				AppControlProfile = $Appprofile;


				OSDAdapter0Name = 'NIC1';


				OSDAdapter0MacAddress = $Macaddress.ToUpper();


				OSDAdapter0IPAddressList = $IP;


				OSDAdapter0SubnetMask = $Subnet;


				OSDAdapter0Gateways = $Gateway;


				OSDAdapter0DNSServerList = $DNSSrv;


				DNSSuffixSearchOrder = $DNSSuf;


				OSDAdapter0EnableDHCP = 'FALSE';


				OSDAdapter0EnableLMHOSTS = 'FALSE';


				OSDAdapter0EnableWINS = 'FALSE';


				OSDAdapter0TcpipNetbiosOptions = '1';


				OSDPartitions = $OSDPartitions;


				OSDPartitions0TYPE = $OSDPartitions0TYPE;


				OSDPartitions0FILESYSTEM = $OSDPartitions0FILESYSTEM;


				OSDPartitions0BOOTABLE = $OSDPartitions0BOOTABLE;


				OSDPartitions0QUICKFORMAT = $OSDPartitions0QUICKFORMAT;


				OSDPartitions0VOLUMENAME = $OSDPartitions0VOLUMENAME;


				OSDPartitions0SIZE = $OSDPartitions0SIZE;


				OSDPartitions0SIZEUNITS = $OSDPartitions0SIZEUNITS;


				OSDPartitions0VOLUMELETTERVARIABLE = $OSDPartitions0VOLUMELETTERVARIABLE;


				DoNotCreateExtraPartition = $DoNotCreateExtraPartition;


				WipeDisk = $WipeDisk;


				SkipComputerName = 'YES';


				SkipTaskSequence = 'YES';


				NTPServers = $NTPServers;


				APP_NET35 = $App_NET35;


				APP_NET462 = $App_NET462;


				APP_CTM = $App_CTM;


				APP_NETBCKP = $App_NETBCKP;


				APP_ORA_11_X64 = $App_ORA_11_x64;


				APP_ORA_11_X86 = $App_ORA_11_x86;


				AdminIP = $AdminIP;


			} | Out-Null


			


			


			$Result = Get-MDTComputer -macAddress $Macaddress


			If ($Result -ne $null)


			{


				$Output_richtextbox.ForeColor = [Drawing.Color]::Green


				$Output_richtextbox.AppendText("`n##################")


				$Output_richtextbox.AppendText("`nRecord Submitted to MDT OK with the following Details")


				$Output_richtextbox.AppendText("`n##################")


				$Output_richtextbox.AppendText("`nHost is: $Hostname")


				$Output_richtextbox.AppendText("`nMAC is $MACAddress")


				$Output_richtextbox.AppendText("`nDOM is: $Domain")


				$Output_richtextbox.AppendText("`nENV is: $Environment")


				$Output_richtextbox.AppendText("`nDESC is: $Description")


				$Output_richtextbox.AppendText("`nOS is: $Operatingsystem")


				$Output_richtextbox.AppendText("`nDataCenter is: $TaskSequence")


				$Output_richtextbox.AppendText("`nAppProfile is: $APPProfile")


				$Output_richtextbox.AppendText("`nIP is: $IP")


				$Output_richtextbox.AppendText("`nMask is: $Subnet")


				$Output_richtextbox.AppendText("`nGateway is: $Gateway")


				$Output_richtextbox.AppendText("`nDNS is: $DNSSrv")


				$Output_richtextbox.AppendText("`nReqNo is: $RequestNo")


				$Output_richtextbox.AppendText("`nDisk Size is: $DiskSize$DiskUnit")


				$Output_richtextbox.AppendText("`nDisk Wipe is: $WipeDisk")


				$Output_richtextbox.AppendText("`nUEFI is: $UEFI")


				$Output_richtextbox.AppendText("`nAPP_NET35 is: $App_NET35")


				$Output_richtextbox.AppendText("`nAPP_NET462 is: $App_NET462")


				$Output_richtextbox.AppendText("`nAPP_CTM is: $App_CTM")


				$Output_richtextbox.AppendText("`nAPP_NETBCKP is: $App_NETBCKP")


				$Output_richtextbox.AppendText("`nAPP_ORA_11_X64 is: $App_ORA_11_x64")


				$Output_richtextbox.AppendText("`nAPP_ORA_11_X86 is: $App_ORA_11_x86")


				$Output_richtextbox.AppendText("`nAdminIP is: $AdminIP")


			}


			Else


			{


				$Output_richtextbox.ForeColor = [Drawing.Color]::Red


				$Output_richtextbox.AppendText("`n")


				$Output_richtextbox.AppendText("`n$Hostname Submitted to MDT FAILED")


			}


		}


		Else


		{


			$Output_richtextbox.Text = $Output_richtextbox.Text.Clear


			$Output_richtextbox.font = "lucida console"


			$Output_richtextbox.ForeColor = [Drawing.Color]::Red


			$Output_richtextbox.AppendText("`n")


			$Output_richtextbox.AppendText("`n##################")


			$Output_richtextbox.AppendText("`nVALIDATION FAILED")


			$Output_richtextbox.AppendText("`nComputer Account or MDT record were not deleted")


			$Output_richtextbox.AppendText("`n##################")


			


		}


	}


	


	


	Else


	{


		$Output_richtextbox.ForeColor = [Drawing.Color]::Red


		$Output_richtextbox.AppendText("`n")


		$Output_richtextbox.AppendText("`n##################")


		$Output_richtextbox.AppendText("`nVALIDATION FAILED")


		$Output_richtextbox.AppendText("`nPLEASE REVIEW FORM")


		$Output_richtextbox.AppendText("`n##################")


	}


}



　


# Exit Button Action


$Exit_button_Click = {


	$terminateScript = $true


	$SGBM_Build_FrontEnd.Close()


	return $terminateScript


}



# Query Button Action


$Query_button_Click = {


	


	# Clear Textbox


	$Output_richtextbox.Text = $Output_richtextbox.Text.Clear


	$Output_richtextbox.font = "lucida console"


	


	$Hostname = $Hostname_textbox.Text


	$MACAddress = $MACAddress_textbox.Text


	


	#Check for Existing Duplicate Entries


	$MDTComputers = Get-MDTComputer | Where-Object { $_.MacAddress -eq $MACAddress }


	


	If ($MDTComputers -ne $null)


	{


		foreach ($C in $MDTComputers)


		{


			$ID = $C.ID


			$Output_richtextbox.ForeColor = [Drawing.Color]::Red


			$Output_richtextbox.AppendText("`n")


			$Output_richtextbox.AppendText("`nServer Exists in MDT with ID: $ID")


		}


	}


	Else


	{


		$Output_richtextbox.ForeColor = [Drawing.Color]::Green


		$Output_richtextbox.AppendText("`n")


		$Output_richtextbox.AppendText("`nNo record in MDT with MAC:  $MACAddress")


	}


	


}



　


　


# Form Actions



$MACAddress_textbox_TextChanged = {


	#TODO: Place custom script here


	If ($MACAddress_textbox.Text -match '^([0-9a-fA-F]{2}[:-]{0,1}){5}[0-9a-fA-F]{2}$')


	{


		$MACAddress_textbox.BackColor = 'PaleGreen'


	}


	else


	{


		$MACAddress_textbox.BackColor = 'LightSalmon'


	}


}



$IP_textbox_TextChanged = {


	


	if ($IP_textbox.Text -match "^(([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])\.){3}([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])$")


	{


		Validate-IPDetails


	}


	else


	{


		$IP_textbox.BackColor = 'LightSalmon'


	}


}



$Subnet_textbox_TextChanged = {


	


	if ($Subnet_textbox.Text -match "^(([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])\.){3}([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])$")


	{


		Validate-IPDetails


	}


	else


	{


		$Subnet_textbox.BackColor = 'LightSalmon'


	}


}



$Gateway_textbox_TextChanged = {


	


	if ($Gateway_textbox.Text -match "^(([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])\.){3}([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])$")


	{


		Validate-IPDetails


	}


	else


	{


		$Gateway_textbox.BackColor = 'LightSalmon'


	}


}



$AdminIP_textbox_TextChanged = {


	if ($AdminIP_textbox.Text -match "^(([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])\.){3}([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])$")


	{


		#Validate-IPDetails


		$AdminIP_textbox.BackColor = 'PaleGreen'


	}


	else


	{


		$AdminIP_textbox.BackColor = 'LightSalmon'


	}


}



$DiskUnit_combobox_SelectedIndexChanged = {


	


	# Check if % has been specified with a value > 100


	[int]$DiskSize = $DiskSize_textbox.Text


	If ($DiskUnit_combobox.Text -eq "%" -and $DiskSize -gt "100")


	{


		$DiskSize_textbox.BackColor = 'LightSalmon'


	}


	else


	{


		$DiskSize_textbox.BackColor = 'PaleGreen'


	}


}



$DiskSize_textbox_TextChanged = {


	# Check if % has been specified with a value > 100


	[int]$DiskSize = $DiskSize_textbox.Text


	If ($DiskUnit_combobox.Text -eq "%" -and $DiskSize -gt "100")


	{


		$DiskSize_textbox.BackColor = 'LightSalmon'


	}


	else


	{


		$DiskSize_textbox.BackColor = 'PaleGreen'


	}


}



$Environment_combobox_TextChanged={


	$DNSSuffix_textbox.Text = $Environment_combobox.SelectedItem.DNSSUF


	$Environment_combobox.BackColor = 'PaleGreen'


}



$Location_combobox_TextChanged={


	$DNSSrv_textbox.Text = $Location_combobox.SelectedItem.DNSSERV


	$NTPServers_textbox.Text = $Location_combobox.SelectedItem.NTPSERV


	$Location_combobox.BackColor = 'PaleGreen'


}



$Hostname_textbox_TextChanged={


	


	$Hostname_textbox.BackColor = 'PaleGreen'


}



$APPProfile_combobox_TextChanged={


	$APPProfile_combobox.BackColor = 'PaleGreen'


	function Clear-Checks


	{


		$NET35_checkbox.Checked = $false


		$NET462_checkbox.Checked = $false


		$ControlM_checkbox.Checked = $false


		$NetBackup_checkbox.Checked = $false


		$Oracle11x64_checkbox.Checked = $false


		$Oracle11X86_checkbox.Checked = $false


	}


	if ($APPProfile_combobox.Text -eq "Database")


	{


		#first clear all checks


		Clear-Checks


		#then set checks


		$ControlM_checkbox.Checked = $true


		$NetBackup_checkbox.Checked = $true


	}


	elseif ($APPProfile_combobox.Text -eq "Batch")


	{


		#first clear all checks


		Clear-Checks


		#then set checks


		$ControlM_checkbox.Checked = $true


	}


	else


	{


		Clear-Checks


	}


}



$Domain_combobox_TextChanged={


	$Domain_combobox.BackColor = 'PaleGreen'


}



$OS_combobox_TextChanged={


	$OS_combobox.BackColor = 'PaleGreen'


}



$Description_textbox_TextChanged={


	$Description_textbox.BackColor = 'PaleGreen'


}



$OS_combobox_SelectedIndexChanged = {


}



$Environment_combobox_SelectedIndexChanged = {


}



$DBServerInstance_label_Click = {


}



$DBServerInstance_label_Click = {


}



$Output_richtextbox_TextChanged = {


}



$DBServerInstance_label_Click = {


}



$Hostname_Label_Click = {


}



$Domain_label_Click = {


}



$SGBMLogo_picturebox_Click = {


}



$IP_label_Click = {


}



$Subnet_label_Click = {


}



$Gateway_Label_Click = {


}



$IP_label_Click = {


}



$Subnet_label_Click = {


}



$Description_label_Click = {


}



$Description_label_Click = {


}



$Gateway_Label_Click = {


}



$Subnet_label_Click = {


}



$Subnet_label_Click = {


}



$Hostname_Label_Click = {


}



$Hostname_Label_Click = {


}



$Domain_label_Click = {


}



$SGBMLogo_picturebox_Click = {


}



$IP_label_Click = {


}



$Subnet_label_Click = {


}



$Gateway_Label_Click = {


}



$Description_label_Click = {


}


$groupbox1_Enter={


	#TODO: Place custom script here


	


}


 
