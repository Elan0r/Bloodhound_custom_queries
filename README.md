# Bloodhound_custom_queries
Custom Queries for Bloodhound AD

My set of custom queries.

### Neo4j queries as addition

via http://localhost:7474/browser/

or Neo4j Browser App.

Usefull queries which can be exported to .csv:

-----------

Check for passwords in User Description.
```cypher
MATCH (u:User) return u.name, u.displayname, u.description
```
or
```cypher
MATCH (u:User) WHERE NOT u.description=[] return u.name, u.displayname, u.description
```

-----------

Check for outdated Systems; Windows 8 and 8.1 (JAN 2023)/ Server 2012 (OKT 2023).
```cypher
MATCH (n:Computer) WHERE n.operatingsystem =~ "(?i).*(2000|2003|2008|2012|xp|vista|7|8|me|98|95).*" RETURN n.name, n.operatingsystem
```

-----------

Check for Computers without LAPS.
```cypher
MATCH (c:Computer) WHERE c.haslaps=FALSE RETURN c.name, c.haslaps, c.domain
```
or
```cypher
MATCH (c:Computer) WHERE c.haslaps=FALSE RETURN c.name, c.haslaps, c.enabled, c.domain
```

-----------

Password Never Expires!
```cypher
MATCH (m:User) WHERE m.pwdneverexpires=TRUE RETURN m.name, m.pwdneverexpires
```
or
```cypher
MATCH (m:User) WHERE m.pwdneverexpires=TRUE RETURN m.name, m.pwdneverexpires, m.pwdlastset
```

-----------

Check Users and where they are admin to.
```cypher
MATCH p=(m:User)-[r:AdminTo]->(n:Computer) Return m.name, n.name
```
or
```cypher
MATCH p=(m:User)-[r:AdminTo]->(n:Computer) where m.admincount = false Return m.name, n.name
```

-----------

Find All Users with an SPN. (Kerberoastable)
```cypher
MATCH (n:User)WHERE n.hasspn=true RETURN n.name, n.pwdlastset
```
Find all Kerberoastable Users with passwords last set less than 5 years ago.
Kerberoastable User and when PW last set in epochtime -> use a converter https://www.epochconverter.com/batch
```cypher
MATCH (u:User) WHERE u.hasspn=true AND u.pwdlastset < (datetime().epochseconds - (1825 * 86400)) AND NOT u.pwdlastset IN [-1.0, 0.0] RETURN u.name, u.pwdlastset order by u.pwdlastset
```

-----------

Mark all Users of Domain-Admin Group as High Value (more in custom queries)
```cypher
MATCH p=(n:Group)<-[:MemberOf*1..]-(m) WHERE n.objectid =~ "(?i)S-1-5-.*-512" SET m.highvalue = true RETURN m
```

-----------

E-Mail-List from AD
```cypher
MATCH (u:User) WHERE NOT u.email=[] RETURN u.name,u.email
```

-----------

free passwords?
```cypher
MATCH (u:User) WHERE NOT u.unixpassword=[] return u.name, u.displayname, u.unixpassword
```

-----------

User never Logged in but enabled
```cypher
MATCH (n:User) WHERE n.lastlogontimestamp=-1.0 AND n.enabled=TRUE RETURN n.name, n.lastlogontimestamp
```


-----------

###AFTER Crackhound
User has Plaintext and is enabled.
```cypher
MATCH (u:User) where u.plaintext=True AND u.enabled=True RETURN u.name, u.plaintextpassword
```


-----------

Manual Crackhound, Username and Domain are all capital letters.
Username Hash and Password needs to be in quotes.
```cypher
MATCH (u:User) where u.name="<BH_USER@DOMAIN>" set u.plaintextpassword="<S3c3t5>" set u.nthash="<HASH>" set u.owned=True set u.plaintext=True return u
```


-----------

User for AZURE?
```cypher
MATCH (u:User) where u.plaintext=True AND u.enabled=True and not u.email = [] RETURN u.email, u.plaintextpassword
```

-----------

Thanks to [@_wald0](https://www.twitter.com/_wald0), [@CptJesus](https://twitter.com/CptJesus), [@harmj0y](https://twitter.com/harmj0y), [@BloodhoundAD](https://github.com/BloodHoundAD/BloodHound) for the tool itself and [@hausec](https://github.com/hausec) and [@ly4k](https://github.com/ly4k) for his queries as template and of course [certipy](https://github.com/ly4k/certipy).
