# LDAP

## ldapTemplate使用
### 新增数据
```
String username = "test02";
String dn = "cn=" + username + ",ou=users,ou=system";

Attributes attrs = new BasicAttributes();
BasicAttribute ocattr = new BasicAttribute("objectclass");
ocattr.add("top");
ocattr.add("inetOrgPerson");
ocattr.add("person");
ocattr.add("organizationalPerson");

attrs.put(ocattr);
attrs.put("cn", "test02");
attrs.put("sn", "test02");
attrs.put("displayName", "test02 test02");
attrs.put("givenName", "test02");
attrs.put("mail", "test02@xxxxxx");

ldapTemplate.bind(dn, null, attrs);
```
### 删除数据
```
String dn = "cn=" + username + ",ou=users,ou=system";
ldapTemplate.unbind(dn);
```
删除数据必须先检查一下是否存在，否则会抛出未找到指定dn的异常。

### 账户密码校验
```
AndFilter filter = new AndFilter();
filter.and(new EqualsFilter("objectclass", "person")).and(new EqualsFilter("cn", username));
boolean authResult = ldapTemplate.authenticate(LdapUtils.emptyLdapName(), filter.toString(), password);
```

### 搜索
lookup用于查询已知存在的dn，如果不存在，将会抛出如下异常。
```
ERR_648 Invalid search base cn=test02,ou=users,ou=system]; nested exception is javax.naming.NameNotFoundException: [LDAP: error code 32 - NO_SUCH_OBJECT: failed for MessageType : SEARCH_REQUEST
Message ID : 4
    SearchRequest
        baseDn : 'cn=test02,ou=users,ou=system'
        filter : '(objectClass=*)'
        scope : base object
        typesOnly : false
        Size Limit : no limit
        Time Limit : no limit
        Deref Aliases : deref Always
        attributes : 
org.apache.directory.api.ldap.model.message.SearchRequestImpl@95632725
```

查询未知dn，如果不存在，返回空列表而不是抛出异常。
```
AndFilter filter = new AndFilter();
filter.and(new EqualsFilter("cn", username));
List list = ldapTemplate.search("", filter.encode(), (Attributes attrs) -> {return attrs.get("cn").get();});
```

按照合适条件，获取所有用户列表
```
AndFilter filter = new AndFilter();
filter.and(new EqualsFilter("objectclass", "person"));  
List list = ldapTemplate.search("", filter.encode(), (Attributes attrs) -> {return attrs.get("cn").get();});

```

