# <a name="headin">VBScript to check for AD users password expire</a>
<a href="https://github.com/cosmingherghel/cosmingherghel.github.io" class="btn btn-github"><span class="icon"></span>View on GitHub</a>

```.vbs                    

##############Description###############
#Script to query all AD user Objects and check for password expiery date.
#If the pasword will expire after 4 days post user Object details to a PHP 
#script that sends the notiffication.
#The script uses the HTTP POST to PHP request method as integration of 
#direct email message management module of PS may fail with some smtp servers 
#with hightened security. 
########################################

##############Variables#################                       
$notificationstartday = 4              
$DN = "DC=my_dom,DC=my_dom,DC=com"  
########################################            

#############Post Method##################
function postToPHP($mailBody1,$mailBody2,$mailBody3,$usermailaddress,$delta)
{
   $postParams = @{"body1"=$mailBody1 ;"body2"=$mailBody2 ;"body3"=$mailBody3 ;"address"=$usermailaddress; "delta"=$delta}
   $req = Invoke-WebRequest -Uri ("http://script_hosting_server/notiff.php") -Method POST -Body $postParams 
}
######################################## 
                    
##############Main######################            
$domainPolicy = Get-ADDefaultDomainPasswordPolicy            
$passwordexpirydefaultdomainpolicy = $domainPolicy.MaxPasswordAge.Days -ne 0            
           
if($passwordexpirydefaultdomainpolicy)            
{            
    $defaultdomainpolicyMaxPasswordAge = $domainPolicy.MaxPasswordAge.Days                  
}            
            
foreach ($user in (Get-ADUser -SearchBase $DN -Filter * -properties mail))            
{            
    $samaccountname = $user.samaccountname            
    $PSO= Get-ADUserResultantPasswordPolicy -Identity $samaccountname             
    if ($PSO -ne $null)            
    {                         
        $PSOpolicy = Get-ADUserResultantPasswordPolicy -Identity $samaccountname            
        $PSOMaxPasswordAge = $PSOpolicy.MaxPasswordAge.days            
        $pwdlastset = [datetime]::FromFileTime((Get-ADUser -LDAPFilter "(&(samaccountname=$samaccountname))" -properties pwdLastSet).pwdLastSet)            
        $expirydate = ($pwdlastset).AddDays($PSOMaxPasswordAge)            
        $delta = ($expirydate - (Get-Date)).Days            
        $comparionresults = (($expirydate - (Get-Date)).Days -le $notificationstartday) -AND ($delta -ge 1)            
        $expired = (($expirydate - (Get-Date)).Days -le $notificationstartday) -AND ($delta -eq 0)
        if ($comparionresults)            
        {            
            $mailBody1 = "Dear " + $user.GivenName + ","            
            if ($delta -eq 1 -Or $delta -lt 1 )
                {
                    $mailBody2 = "Your password for my_domain domain will expire after " + $delta + " days. You need to change your password immediately."
                } else 
                    {
                      $mailBody2 = "Your password for my_domain domain will expire after " + $delta + " days. You will need to change your password as soon as possible."            
                    }
            $mailBody3 = "IT Department"            
            $usermailaddress = $user.mail           
            postToPHP $mailBody1 $mailBody2 $mailBody3 $usermailaddress $delta
        } elseif ($expired)
              {
                $mailBody1 = "Dear " + $user.GivenName + ","
                $mailBody2 = "Your password for INTRA.SATA.COM has expired. You need to change your password before your next log in."
                $mailBody3 = "IT Department"
                $usermailaddress = $user.mail            
               postToPHP $mailBody1 $mailBody2 $mailBody3 $usermailaddress $delta
              }                       
    }            
    else            
    {            
        if($passwordexpirydefaultdomainpolicy)            
        {            
            $pwdlastset = [datetime]::FromFileTime((Get-ADUser -LDAPFilter "(&(samaccountname=$samaccountname))" -properties pwdLastSet).pwdLastSet)            
            $expirydate = ($pwdlastset).AddDays($defaultdomainpolicyMaxPasswordAge)            
            $delta = ($expirydate - (Get-Date)).Days            
            $comparionresults = (($expirydate - (Get-Date)).Days -le $notificationstartday) -AND ($delta -ge 1)            
            $expired = (($expirydate - (Get-Date)).Days -le $notificationstartday) -AND ($delta -eq 0)
          
            if ($comparionresults)            
            {            
              $mailBody1 = "Dear " + $user.GivenName + ","             
              $delta = ($expirydate - (Get-Date)).Days            
              if ($delta -eq 1 -Or $delta -lt 1 )
              {
                  $mailBody2 = "Your password for my_domain domain will expire after " + $delta + " days. You need to change your password immediately."
               } else 
                     {
                        $mailBody2 = "Your password for my_domain domain will expire after " + $delta + " days. You will need to change your password as soon as possible."            
                      }
                
            $mailBody3 = "Sata Bank IT Department"            
            $usermailaddress = $user.mail            
            postToPHP $mailBody1 $mailBody2 $mailBody3 $usermailaddress $delta   
            } elseif ($expired)
              {
                $mailBody1 = "Dear " + $user.GivenName + ","
                $mailBody2 = "Your password for my_domain has expired. You need to change your password before your next log in."
                $mailBody3 = "IT Department"
                $usermailaddress = $user.mail            
               postToPHP $mailBody1 $mailBody2 $mailBody3 $usermailaddress $delta
              }            
            
        }            
    }            
}
```
[Back to top](#headin)
