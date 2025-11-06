## OCP Deployment Devops Container Image Build Automation

### Fapolicyd rule for various clients 
If deploying or running on RHEL8+ with fapolicyd running on it, you will need to ensure that rules are properly added for the various clients being installed on the bastion. Use the following steps to troubleshoot permission denial errors and add rules based on errors. 

To troubleshoot fapolicyd errors follow these steps 
1. stop the service using `systemctl stop fapolicyd.service`

2. start the service in debug mode in backgroud using `fapolicyd --debug-deny 2> fapolicy.output &`

3. in another terminal rerun the command that was denied so that the denial message can be added to the `fapolicy.output` created above  

4. look for the deny audit log using `cat fapolicy.output | grep 'deny_audit'` (example rule can be `rule=23 dec=deny_audit perm=open auid=1000 pid=12762 exe=/usr/local/aws-cli/v2/dist/aws : path=/usr/local/aws-cli/v2/dist/libz.so.1 ftype=application/x-sharedlib trust=0`)

5. locate the rule file containing the deny rule by using `grep -r "deny_audit perm=open" /etc/fapolicyd/rules.d/` assuming the located rule in step 4 has the string `deny_audit perm=open` in it 

6. add an allow rule to the file lexicographycally preceeding the deny rule file. If the deny rule located in step 5 above is in a file with name starting with 7- then the new rule need to be added to a file with name 6-. Add the rule to the file using correct exe and path or dir (e.g. allow perm=open exe=/usr/bin/python3.9 trust=1 : dir=/home/myuser/.ansible/tmp/ trust=0). See sample rules section below for working samples. 

7. validate the rule using `fagenrules --check`

8. load the rule using `fagenrules --load`

9. list the loaded rules using `fapolicyd-cli --list`

10. stop the fapolicyd debug mode using fg + ctnl C command sequence or kill <bg-id> with bg-id being the id of the background process

11. start the fapolicyd service using `systemctl start fapolicyd`

12. check the service status using `systemctl status fapolicyd`


### Fapolicyd rule samples for various clients 

#### Fapolicyd rule samples for various directories 
```
allow perm=open all : dir=/usr/bin
allow perm=open all : dir=/home
allow perm=open all : dir=/root
allow perm=execute exe=/usr/bin/bash trust=1 : dir=/home trust=0
allow perm=execute exe=/usr/bin/bash trust=1 : dir=/root trust=0
```

#### Fapolicyd rule samples for oc client
```
allow perm=execute exe=/usr/bin/oc trust=1 : all
allow perm=execute exe=/usr/bin/oc : all
allow perm=execute exe=/usr/bin/bash : path=/usr/bin/oc ftype=application/x-executable trust=0
```

#### Fapolicyd rule samples for terraform client
```
allow perm=open all : dir=/usr/bin
allow perm=open exe=/usr/bin/terraform all trust=1 : all
allow perm=execute exe=/usr/bin/bash : path=/usr/bin/terraform ftype=application/x-executable trust=0
allow perm=open exe=/usr/bin/terraform trust=1 : dir=/tmp/ trust=0
```

#### Fapolicyd rule samples for awscli client
```
allow perm=execute exe=/usr/bin/bash : path=/usr/bin/aws ftype=application/x-executable trust=0
allow perm=open exe=/usr/bin/aws trust=1 : dir=/home/ trust=0

allow perm=open exe=/usr/local/aws-cli/v2/dist/aws : dir=/usr/local/aws-cli/v2/dist/ all ftype=application/x-sharedlib trust=0
allow perm=open exe=/usr/local/aws-cli/v2/dist/aws : dir=/usr/local/aws-cli/v2/dist/lib-dynload/ all ftype=application/x-sharedlib trust=0
allow perm=execute exe=/usr/local/aws-cli/v2/dist/aws trust=1 : dir=usr/local/aws-cli/v2/dist/aws trust=0
allow perm=execute exe=/usr/bin/aws trust=1 : dir=/usr/bin/aws trust=0
```
