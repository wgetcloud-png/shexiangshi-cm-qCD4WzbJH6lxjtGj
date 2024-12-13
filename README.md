
使用 SDDC Manager 中的“密码管理”功能可以统一[管理 VCF 环境中组件的用户密码](https://github.com)，比如更新（Update）、轮换（Rotate）以及修复（Remediate）组件的密码等，您还可以创建密码轮换调度任务，以防止因遗忘或其他原因导致密码过期及组件中断，进而影响业务。


[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241212095902508-2044063496.png)](https://github.com)


使用 SoS 实用程序可以检查 VCF 环境中组件的用户密码状态，比如最后一次修改日期、过期日期以及过期剩余时间等，如下所示。



```
vcf@vcf-mgmt01-sddc01 [ ~ ]$ sudo /opt/vmware/sddc-support/sos --password-health
[sudo] password for vcf
Welcome to Supportability and Serviceability(SoS) utility!
Performing SoS operation for vcf-mgmt01 domain components
Health Check : /var/log/vmware/vcf/sddc-support/healthcheck-2024-12-07-12-29-31-149728
Health Check log : /var/log/vmware/vcf/sddc-support/healthcheck-2024-12-07-12-29-31-149728/sos.log
NOTE : The Health check operation was invoked without --skip-known-host-check, additional identity checks will be included for Connectivity Health, Password Health and Certificate Health Checks because of security reasons.

SDDC Manager : vcf-mgmt01-sddc01.mulab.local                                                                                
+-------------------------+-----------+
|          Stage          |   Status  |
+-------------------------+-----------+
|         Bringup         | Completed |
| Management Domain State | Completed |
+-------------------------+-----------+
+--------------------+---------------+
|     Component      |    Identity   |
+--------------------+---------------+
|    SDDC-Manager    | 192.168.32.70 |
| Number of Servers  |       4       |
+--------------------+---------------+
Password Expiry Status : GREEN                                                                                 
+-----+-----------------------------------------+---------------------------+-------------------+--------------+-----------------+-------+
| SL# |                Component                |            User           | Last Changed Date | Expiry Date  | Expires in Days | State |
+-----+-----------------------------------------+---------------------------+-------------------+--------------+-----------------+-------+
|  1  |   ESXI : vcf-mgmt01-esxi01.mulab.local  | svc-vcf-vcf-mgmt01-esxi01 |    Dec 02, 2024   |    Never     |      Never      | GREEN |
|     |                                         |            root           |    Dec 02, 2024   |    Never     |      Never      | GREEN |
|  2  |   ESXI : vcf-mgmt01-esxi02.mulab.local  | svc-vcf-vcf-mgmt01-esxi02 |    Dec 02, 2024   |    Never     |      Never      | GREEN |
|     |                                         |            root           |    Dec 02, 2024   |    Never     |      Never      | GREEN |
|  3  |   ESXI : vcf-mgmt01-esxi03.mulab.local  | svc-vcf-vcf-mgmt01-esxi03 |    Dec 02, 2024   |    Never     |      Never      | GREEN |
|     |                                         |            root           |    Dec 02, 2024   |    Never     |      Never      | GREEN |
|  4  |   ESXI : vcf-mgmt01-esxi04.mulab.local  | svc-vcf-vcf-mgmt01-esxi04 |    Dec 02, 2024   |    Never     |      Never      | GREEN |
|     |                                         |            root           |    Dec 02, 2024   |    Never     |      Never      | GREEN |
|  5  |    NSX : vcf-mgmt01-nsx01.mulab.local   |           admin           |    Dec 07, 2024   | Mar 07, 2025 |     90 days     | GREEN |
|     |                                         |            root           |    Dec 07, 2024   | Mar 07, 2025 |     90 days     | GREEN |
|     |                                         |           audit           |    Dec 07, 2024   | Mar 07, 2025 |     90 days     | GREEN |
|  6  |   SDDC : vcf-mgmt01-sddc01.mulab.local  |            vcf            |    Dec 07, 2024   | Dec 07, 2025 |     365 days    | GREEN |
|     |                                         |            root           |    Dec 07, 2024   | Mar 07, 2025 |     90 days     | GREEN |
|     |                                         |           backup          |    Dec 07, 2024   | Dec 07, 2025 |     365 days    | GREEN |
|  7  | vCenter : vcf-mgmt01-vcsa01.mulab.local |            root           |    Dec 07, 2024   | Mar 07, 2025 |     89 days     | GREEN |
+-----+-----------------------------------------+---------------------------+-------------------+--------------+-----------------+-------+

Legend:

 GREEN - No attention required, health status is NORMAL
 YELLOW - May require attention, health status is WARNING
 RED - Requires immediate attention, health status is CRITICAL


Health Check completed successfully for : [VCF-SUMMARY, PASSWORD-CHECK]                                                                                
vcf@vcf-mgmt01-sddc01 [ ~ ]$
```

根据上面所输出的结果，能够很清楚的了解各个组件用户密码的状态，不过，你可能想知道我能不能重新调整一下这些组件的默认“密码策略”？比如密码过期、密码复杂度以及账户锁定等。答案是肯定的！首先，让我们参考[《Information Security and Access of Identity and Access Management for VMware Cloud Foundation》](https://github.com)产品文档，先来了解一下 VCF 环境中组件的默认密码策略。


 



## 一、密码过期策略





| **组件** | **级别** | **参数设置** | **默认** | **描述** | **备注** |
| --- | --- | --- | --- | --- | --- |
| **ESXi 主机** | 本地用户 | Security.PasswordMaxDays | 99999 (never) | 设置多少天密码过期。 | 您可以使用 vSphere Client 或 Host Client 中的高级系统设置按主机管理密码过期策略。您可以修改每个 ESXi 主机上的配置设置，以优化设置并遵守组织的策略和法规标准。 |
| **vCenter Server** | 全局 | Maximum (days) | 90 | 设置最大多少天密码过期。 | 您可以按实例管理密码过期策略。您可以修改每个 vCenter Server 实例上的配置设置，以优化设置并遵守组织的策略和法规标准。 |
| Minimum (days) | 0 | 设置最小多少天密码过期。 |
| Warning | 7 | 设置密码过期前多少天警告。 |
| 本地用户 | Password Expires | Yes | 设置 root 密码是否过期。 |
| Password validity | 90 | 设置多少天密码过期。 |
| Email for expiration warning | \- | 设置密码过期警告的电子邮件。 |
| Warning (days) | 7 | 设置密码过期前多少天警告。 |
| Single Sign\-On | Maximum lifetime | 90 | 设置多少天密码过期。 | 您可以管理每个内置身份提供程序域的 vCenter Single Sign\-On 密码过期策略。密码过期策略仅适用于 vCenter Single Sign\-On 内置身份提供程序域（例如 vsphere.local）中的用户帐户。该策略不适用于本地系统账户或域的默认管理员账户（例如，administrator@vsphere.local）。您可以修改 vCenter Single Sign\-On 身份提供程序域的配置设置，以优化设置并遵守组织的策略和法规标准。 |
| **NSX \+ NSX Edge** | 本地用户 | maxdays | 90 | 设置最大多少天密码过期。 | 通过使用 NSX Local Manager 集群和 NSX Edge 节点上内置 NSX 帐户的 API，您可以按用户管理 NSX 密码过期策略。您可以修改每个 NSX Local Manager 集群和每个 NSX Edge 节点上的配置，以优化设置并遵守组织的策略和法规标准。 |
| **SDDC Manager** | 本地用户 | maxdays | 90 | 设置最大多少天密码过期。 （VMware Cloud Foundation 4\.5 及更高版本） | 您可以基于用户管理密码过期策略。您可以修改用户的配置，以优化设置并遵守组织的策略和法规标准。 |
| 365 | 设置最大多少天密码过期。 （VMware Cloud Foundation 4\.4 及更高版本） |
| mindays | 0 | 设置最小多少天密码过期。 |
| warndays | 7 | 设置密码过期前多少天警告。 |


 



## 二、密码复杂性策略





| **组件** | **级别** | **参数设置** | **默认** | **描述** | **备注** |
| --- | --- | --- | --- | --- | --- |
| **ESXi 主机** | 本地用户 | Security.PasswordQualityControl | `retry=3 min=disabled,disabled,disabled,7,7` | 密码设置或更新的重设次数，值为3表示设置密码时如果密码不符合上述要求的重试次数为3次。 字符类别和密码短语最小长度要求。不允许使用包含一种或两种类别字符的密码，也不允许使用密码短语，因为前三项已停用。使用三种和四种字符类别的密码需要 7 个字符。 | 密码开头的大写字母不算入使用的字符类别数。密码结尾的数字不算入使用的字符类别数。 您可以使用 vSphere Client 或 Host Client 中的高级系统设置按主机管理密码复杂性策略。您可以编辑和修改配置以优化设置并遵守组织的策略和法规标准。 |
| Security.PasswordHistory | 0 | 设置记住曾设置密码的历史次数，值为0表示不限制。 |
| **vCenter Server** | 本地用户 | dcredit | \-1 | 设置密码应包含的数字字符（如0、1、2）数量，值为\-1表示至少一个。 | 您可以通过按实例使用 /etc/pam.d/system\-password 文件来管理本地用户密码复杂性策略。您可以修改每个 vCenter Server 实例上的配置设置，以优化设置并遵守组织的策略和法规标准。 |
| ucredit | \-1 | 设置密码应包含的大写字母（如A、B、C）数量，值为\-1表示至少一个。 |
| lcredit | \-1 | 设置密码应包含的小写字母（如a、b、c）数量，值为\-1表示至少一个。 |
| ocredit | \-1 | 设置密码应包含的其他字符（如！、@、\#）数量，值为\-1表示至少一个。 |
| minlen | 6 | 设置密码应具有的最小字符数量，值为6表示至少具有六个字符。 |
| difok | 4 | 设置新密码与旧密码相比不同的字符数量，值为4表示至少具有四个字符不一样。 |
| remember | 5 | 设置记住曾设置密码的历史次数，值为5表示新密码不应该是之前设置的5次密码中的任何一个。 |
| Single Sign\-On | Restrict reuse | 5 | 设置记住曾设置密码的历史次数，值为5表示新密码不应该是之前设置的5次密码中的任何一个。 | 您可以管理每个内置身份提供程序域的 vCenter Single Sign\-On 密码过期策略。密码复杂性策略仅适用于 vCenter Single Sign\-On 内置身份提供程序域（例如 vsphere.local）中的用户帐户。该策略不适用于本地系统账户或内置身份提供商的默认管理员账户（例如，administrator@vsphere.local）。 |
| Maximum length | 20 | 设置最大密码长度（字符数）。 |
| Minimum length | 8 | 设置最小密码长度（字符数）。 |
| Special characters | 1 | 设置特殊字符的最少数量。 |
| Alphabetic characters | 2 | 设置最少字母字符数。 |
| Uppercase characters | 1 | 设置最少大写字符数。 |
| Lowercase characters | 1 | 设置最少小写字符数。 |
| Numeric characters | 1 | 设置最小数字字符数。 |
| Identical adjacent characters | 1 | 设置相同相邻字符的最大数量。 |
| **NSX \+ NSX Edge** | 本地用户 | dcredit | \-1 | 设置密码应包含的数字字符（如0、1、2）数量，值为\-1表示至少一个。 | 您可以通过对 NSX Manager 集群和 NSX Edge 节点上的内置 NSX 帐户按节点使用 /etc/pam.d/common\-password 文件来管理密码复杂性策略。您可以修改每个 NSX Manager 节点和每个 NSX Edge 节点上的配置，以优化设置并遵守组织的策略和法规标准。 |
| ucredit | \-1 | 设置密码应包含的大写字母（如A、B、C）数量，值为\-1表示至少一个。 |
| lcredit | \-1 | 设置密码应包含的小写字母（如a、b、c）数量，值为\-1表示至少一个。 |
| ocredit | \-1 | 设置密码应包含的其他字符（如！、@、\#）数量，值为\-1表示至少一个。 |
| minlen | 15 | 设置密码应具有的最小字符数量，值为15表示至少具有十五个字符。 |
| difok | 0 | 设置新密码与旧密码相比不同的字符数量，值为0表示不限制。 |
| retry | 3 | 密码设置或更新的重设次数，值为3表示设置密码时如果密码不符合上述要求的重试次数为3次。 |
| **SDDC Manager** | 本地用户 | dcredit | \-1 | 设置密码应包含的数字字符（如0、1、2）数量，值为\-1表示至少一个。 | 您可以使用 /etc/pam.d/system\-password 文件管理密码复杂性策略。您可以编辑和修改配置以优化设置并遵守组织的策略和法规标准。 |
| ucredit | \-1 | 设置密码应包含的大写字母（如A、B、C）数量，值为\-1表示至少一个。 |
| lcredit | \-1 | 设置密码应包含的小写字母（如a、b、c）数量，值为\-1表示至少一个。 |
| ocredit | \-1 | 设置密码应包含的其他字符（如！、@、\#）数量，值为\-1表示至少一个。 |
| minlen | 8 | 设置密码应具有的最小字符数量，值为15表示至少具有十五个字符。 |
| minclass | 4 | 设置密码必须使用的最小字符类型数（例如，大写、小写、数字等）。 |
| difok | 4 | 设置新密码与旧密码相比不同的字符数量，值为4表示至少具有四个字符不一样。 |
| retry | 3 | 密码设置或更新的重设次数，值为3表示设置密码时如果密码不符合上述要求的重试次数为3次。 |
| maxsequence | 0 | 设置密码单个字符可以重复的最大次数，值为0表示不限制。 |
| remember | 5 | 设置记住曾设置密码的历史次数，值为5表示新密码不应该是之前设置的5次密码中的任何一个。 |


 



## 三、账户锁定策略





| **组件** | **级别** | **参数设置** | **默认** | **描述** | **备注** |
| --- | --- | --- | --- | --- | --- |
| **ESXi 主机** | 本地用户 | Security.AccountLockFailures | 5 | 设置帐户被锁定之前的最大身份验证失败次数。 | SSH 和 API 支持 ESXi 帐户锁定。如果用户尝试使用 SSH 或 API 使用不正确的本地帐户凭据登录，则该帐户将被锁定。DCUI 和 ESXi Shell 不支持帐户锁定。 您可以使用 vSphere Client 或 Host Client 中的高级系统设置按主机管理帐户锁定策略。您可以编辑和修改配置以优化设置并遵守组织的策略和法规标准。 |
| Security.AccountUnlockTime | 900 | 设置帐户处于锁定状态的时间（以秒为单位）。 |
| **vCenter Server** | 本地用户 | deny | 3 | 设置帐户被锁定之前的最大身份验证失败次数。 | 您可以通过按实例使用 /etc/pam.d/system\-auth 文件来管理本地用户帐户锁定策略。您可以修改每个 vCenter Server 实例上的配置设置，以优化设置并遵守组织的策略和法规标准。 |
| unlock\_time | 900 | 设置帐户处于锁定状态的时间（以秒为单位）。 |
| root\_unlock\_time | 300 | 设置 root 帐户处于锁定状态的时间（以秒为单位）。 |
| Single Sign\-On | Maximum number of failed login attempts | 5 | 设置帐户被锁定之前的最大身份验证失败次数。 | 您可以按内置身份提供程序域管理 vCenter Single Sign\-On 帐户锁定策略。您可以编辑和修改配置以优化设置并遵守组织的策略和法规标准。 |
| Time interval between failures | 180 | 设置登录失败的时间（以秒为单位），如 180 秒内连续失败了 5 次才将账户锁定。 |
| Unlock time | 900 | 设置 root 帐户处于锁定状态的时间（以秒为单位）。如果将其设置为 0，则管理员必须显式解锁帐户。 |
| **NSX \+ NSX Edge** | 本地用户（API） | max\-auth\-failures | 5 | 设置帐户被锁定之前的最大身份验证失败次数。 | 您可以使用身份验证策略，可以按 NSX Local Manager 集群的实例和 NSX Edge 节点的节点实例管理账户锁定策略。您可以为 NSX Local Manager 集群和 NSX Edge 节点的 NSX Manager 用户界面和 API 以及命令行界面 （CLI） 配置帐户锁定策略。您可以修改每个 NSX Local Manager 集群和每个 NSX Edge 节点上的配置，以优化设置并遵守组织的策略和法规标准。 |
| lockout\-reset\-period | 180 | 设置登录失败的时间（以秒为单位），如 180 秒内连续失败了 5 次才将账户锁定。 |
| lockout\-period | 900 | 设置帐户处于锁定状态的时间（以秒为单位）。 |
| 本地用户（CLI） | max\-auth\-failures | 5 | 设置帐户被锁定之前的最大身份验证失败次数。 |
| lockout\-period | 900 | 设置帐户处于锁定状态的时间（以秒为单位）。 |
| **SDDC Manager** | 本地用户 | deny | 3 | 设置帐户被锁定之前的最大身份验证失败次数。 | 您可以使用 /etc/pam.d/system\-auth 文件管理帐户锁定策略。您可以编辑和修改配置以优化设置并遵守组织的策略和法规标准。 |
| unlock\_time | 86400 | 设置帐户处于锁定状态的时间（以秒为单位）。 |
| root\_unlock\_time | 300 | 设置 root 帐户处于锁定状态的时间（以秒为单位）。 |


 



## 四、管理密码策略



了解了 VCF 环境中组件的密码策略后，因为合规性或者安全性等要求，可能想要管理这些 VCF 组件的密码策略，因此您可以参考这篇[《VMware Cloud Foundation Operations Guide》](https://github.com)产品文档中的以下内容，然后根据需要执行对应组件的密码策略管理。


* [Configuring Password Expiration Policies in VMware Cloud Foundation](https://github.com)
* [Configuring Password Complexity Policies in VMware Cloud Foundation](https://github.com)
* [Configuring Account Lockout Policies in VMware Cloud Foundation](https://github.com)


根据上面所列出的操作文档可知，要管理这些密码策略可以使用多种方式，最原始的办法就是针对不同的组件的不同密码策略，手动从组件中去修改那些参数，然后逐一完成组件的密码策略调整过程。但是这种方式太过于繁琐了，对于有成百上千台主机的大型环境来说，文档中的另一种方式会更加方便，那就是使用“PowerShell”命令来配置。使用 PowerShell 来配置需要借助以下 PowerShell 模块：


* [PowerShell Module for VMware Cloud Foundation Password Management](https://github.com) (PowerShell Gallery)
* [PowerShell Module for VMware Cloud Foundation Password Management](https://github.com) (Github)


参考文档，然后使用以下命令安装 VCF 密码管理模块。注意，安装使用这个模块之前，请参考这篇“[使用 PowerVCF 连接和管理 VMware Cloud Foundation 环境。](https://github.com)”文章准备这个模块所依赖的其他运行环境。



```
Install-Module -Name VMware.CloudFoundation.PasswordManagement -Scope CurrentUser
Get-Module -Name VMware.CloudFoundation.PasswordManagement -ListAvailable
```

[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241208151634335-722326155.png)](https://github.com)


查看模块所支持的命令选项。



```
Get-Command -Module VMware.CloudFoundation.PasswordManagement
```

[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241208152527996-378000456.png)](https://github.com)


验证环境是否满足运行 PowerShell 模块要求。



```
Test-VcfPasswordManagementPrereq
```

[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241208154904504-793029241.png)](https://github.com)


使用 PowerVCF 连接到 SDDC Manager。



```
Request-VCFToken -fqdn vcf-mgmt01-sddc01.mulab.local -username administrator@vsphere.local -password Vcf521@password
```

[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241208152839832-533592032.png)](https://github.com):[豆荚加速器](https://yirou.org)


使用以下命令获取指定 VCF 版本所有组件的默认密码策略，或者直接将其输出为 JSON 文件。



```
Get-PasswordPolicyDefault -version '5.2.0.0'
Get-PasswordPolicyDefault -generateJson -jsonFile passwordPolicyConfig.json -version '5.2.0.0'
```

[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241208154158755-841351314.png)](https://github.com)


使用以下命令获取 VCF 环境中组件**密码策略**报告。


* 所有工作负载域



```
Invoke-PasswordPolicyManager -sddcManagerFqdn vcf-mgmt01-sddc01.mulab.local -sddcManagerUser administrator@vsphere.local -sddcManagerPass Vcf521@password -sddcRootPass Vcf521@password -reportPath "D:\Reporting" -darkMode -allDomains
```

* 指定工作负载域



```
Invoke-PasswordPolicyManager -sddcManagerFqdn vcf-mgmt01-sddc01.mulab.local -sddcManagerUser administrator@vsphere.local -sddcManagerPass Vcf521@password -sddcRootPass Vcf521@password -reportPath "D:\Reporting" -darkMode -workloadDomain vcf-mgmt01
```

[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241208160522631-1016249883.png)](https://github.com)


使用以下命令获取 VCF 环境中组件**密码轮换**报告。


* 所有工作负载域



```
Invoke-PasswordRotationManager  -sddcManagerFqdn vcf-mgmt01-sddc01.mulab.local -sddcManagerUser administrator@vsphere.local -sddcManagerPass Vcf521@password -sddcRootPass Vcf521@password -reportPath "D:\Reporting" -darkMode -allDomains
```

* 指定工作负载域



```
Invoke-PasswordRotationManager -sddcManagerFqdn vcf-mgmt01-sddc01.mulab.local -sddcManagerUser administrator@vsphere.local -sddcManagerPass Vcf521@password -sddcRootPass Vcf521@password -reportPath "D:\Reporting" -darkMode -workloadDomain vcf-mgmt01
```

[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241208160852947-650075413.png)](https://github.com)


根据 JSON 文件以及报告文件，对 VCF 环境中的所有组件统一配置密码策略。



```
Start-PasswordPolicyConfig -sddcManagerFqdn vcf-mgmt01-sddc01.mulab.local -sddcManagerUser administrator@vsphere.local -sddcManagerPass Vcf521@password -sddcRootPass Vcf521@password -reportPath "D:\Reporting" -policyFile "passwordPolicyConfig.json"
```

检索指定 VCF 组件的本地用户的密码过期策略。



```
Request-LocalUserPasswordExpiration -server vcf-mgmt01-sddc01.mulab.local -user administrator@vsphere.local -pass Vcf521@password -domain vcf-mgmt01 -product vcenterServer -vmName vcf-mgmt01-vcsa01 -guestUser root -guestPassword Vcf521@password -localUser "root"
```

[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241208161406074-527633695.png)](https://github.com)


更新指定 VCF 组件的本地用户密码过期期限（以天为单位）。



```
Update-LocalUserPasswordExpiration -server vcf-mgmt01-sddc01.mulab.local -user administrator@vsphere.local -pass Vcf521@password -domain vcf-mgmt01 -vmName vcf-mgmt01-vcsa01 -guestUser root -guestPassword Vcf521@password -localUser "root","sshuser" -minDays 0 -maxDays 180 -warnDays 14
```

[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241208162438110-5233132.png)](https://github.com)


检索 SDDC Manager 组件的密码过期策略。



```
Request-SddcManagerPasswordExpiration -server vcf-mgmt01-sddc01.mulab.local -user administrator@vsphere.local -pass Vcf521@password  -rootPass Vcf521@password
```

[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241208165105910-1977795866.png)](https://github.com)


更新 SDDC Manager 组件的密码过期策略。



```
Update-SddcManagerPasswordExpiration -server vcf-mgmt01-sddc01.mulab.local -user administrator@vsphere.local -pass Vcf521@password  -rootPass Vcf521@password -minDays 0 -maxDays 400 -warnDays 14
```

[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241208165446115-777959263.png)](https://github.com)


检索 ESXi 组件的密码复杂性策略。



```
Request-EsxiPasswordComplexity -server vcf-mgmt01-sddc01.mulab.local -user administrator@vsphere.local -pass Vcf521@password -domain vcf-mgmt01 -cluster vcf-mgmt01-cluster01
```

[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241208170902491-225733810.png)](https://github.com)


更新 ESXi 组件的密码复杂性策略。



```
Update-EsxiPasswordComplexity -server vcf-mgmt01-sddc01.mulab.local -user administrator@vsphere.local -pass Vcf521@password -domain vcf-mgmt01 -cluster vcf-mgmt01-cluster01 -policy "retry=5 min=disabled,disabled,disabled,8,8" -history 3
```

[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241208171301104-344603170.png)](https://github.com)


检索 ESXi 组件的账户锁定策略。



```
Request-EsxiAccountLockout -server vcf-mgmt01-sddc01.mulab.local -user administrator@vsphere.local -pass Vcf521@password -domain vcf-mgmt01 -cluster vcf-mgmt01-cluster01
```

[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241208172426859-739687858.png)](https://github.com)


更新 ESXi 组件的账户锁定策略。



```
Update-EsxiAccountLockout -server vcf-mgmt01-sddc01.mulab.local -user administrator@vsphere.local -pass Vcf521@password -domain vcf-mgmt01 -cluster vcf-mgmt01-cluster01 -failures 3 -unlockInterval 600
```

[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241208172658061-187620310.png)](https://github.com)


检索 SDDC Manager 管理的指定工作负载域的所有组件的密码轮换设置。



```
Request-PasswordRotationPolicy -server vcf-mgmt01-sddc01.mulab.local -user administrator@vsphere.local -pass Vcf521@password
```

[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241208175033335-427586233.png)](https://github.com)


更新 SDDC Manager 管理的 vCenter Server 组件的密码轮换设置。



```
Update-PasswordRotationPolicy -server vcf-mgmt01-sddc01.mulab.local -user administrator@vsphere.local -pass Vcf521@password -domain vcf-mgmt01 -resource vcenterServer -resourceName vcf-mgmt01-vcsa01.mulab.local -credential SSH -credentialName root -autoRotate enabled -frequencyInDays 90
```

[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241208180311988-1863627881.png)](https://github.com)


