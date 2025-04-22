# LDAP Enumeration From Linux

#### Using LDAP Search <a href="#using-ldap-search" id="using-ldap-search"></a>

```bash
# To Check for connections
ldapsearch -H LDAP://10.10.10.161

# -x for basic authentication -s for scope and nammingContexts to get the FQDN
ldapsearch -H LDAP://10.10.10.161 -x -s base namingContexts

ldapsearch -H LDAP://FOREST.htb.local

#Using  -b for domain and param (objectClass=*) to get all data
ldapsearch -H ldap://10.10.10.161/ -x -b 'DC=htb,DC=local' "(objectClass=*)" 

# To search for Groups
ldapsearch -H ldap://10.10.10.161/ -x -b 'DC=htb,DC=local' "(objectClass=Group)" | grep sAMAccountName
ldapsearch -H ldap://10.10.10.161/ -x -b 'DC=htb,DC=local' "(objectClass=Group)" | grep sAMAccountName | cut -d " " -f2


# To Search for Users
ldapsearch -H ldap://10.10.10.161/ -x -b 'DC=htb,DC=local' "(objectClass=User)" | grep sAMAccountName



```
