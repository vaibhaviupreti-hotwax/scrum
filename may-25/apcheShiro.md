# Apache Shiro — End-to-End Documentation & Integration with Apache OFBiz

---

## Table of Contents

1. [What is Apache Shiro?](#1-what-is-apache-shiro)
2. [Core Architecture & Concepts](#2-core-architecture--concepts)
3. [Authentication](#3-authentication)
4. [Authorization](#4-authorization)
5. [Permissions (WildcardPermission System)](#5-permissions-wildcardpermission-system)
6. [Realms](#6-realms)
7. [Session Management](#7-session-management)
8. [Cryptography](#8-cryptography)
9. [Web Integration](#9-web-integration)
10. [Annotations & Declarative Security](#10-annotations--declarative-security)
11. [Configuration (shiro.ini)](#11-configuration-shiroini)
12. [Apache OFBiz + Shiro Integration](#12-apache-ofbiz--shiro-integration)
13. [OFBiz Security Permission Model (Deep Dive)](#13-ofbiz-security-permission-model-deep-dive)
14. [OFBiz Permission Enforcement Points](#14-ofbiz-permission-enforcement-points)
15. [OFBiz Data Model for Security](#15-ofbiz-data-model-for-security)
16. [Code Examples: Shiro in OFBiz](#16-code-examples-shiro-in-ofbiz)
17. [Best Practices & Tips](#17-best-practices--tips)

---

## 1. What is Apache Shiro?

Apache Shiro (pronounced "shee-roh", Japanese for *castle* 城) is a powerful, easy-to-use Java security framework that handles:

- **Authentication** — Who are you? (Login/Identity verification)
- **Authorization** — What can you do? (Access control)
- **Cryptography** — Secure data handling (hashing, encryption)
- **Session Management** — Managing user sessions (even outside web containers)

> **Current Version:** Shiro 2.1.0 (Feb 2026); 3.0.0-alpha-1 also available. As of Feb 28, 2024, Shiro v1 is superseded by v2.

Shiro's philosophy: *"Security shouldn't be hard"*. It wraps complexity behind clean, intuitive APIs centered on the `Subject` — the currently-executing user.

---

## 2. Core Architecture & Concepts

```
Application Code
      |
      v
   Subject  ←── "Who is currently executing?"
      |
      v
 SecurityManager  ←── Central coordinator (umbrella component)
    /    |    \
   /     |     \
Authenticator  Authorizer  SessionManager
   |               |
Realm(s)       Realm(s)
   |               |
Data Store     Data Store
(LDAP/DB/LDAP)
```

### Key Components

| Component | Role |
|---|---|
| `Subject` | The "current user" — your primary interaction point |
| `SecurityManager` | Coordinates all security operations; configured once |
| `Authenticator` | Handles login logic, delegates to Realms |
| `Authorizer` | Handles access control checks, delegates to Realms |
| `Realm` | Bridge between Shiro and your actual data source |
| `SessionManager` | Creates/manages user sessions |
| `CacheManager` | Caches auth data for performance |
| `Cryptography` | Password hashing and data encryption utilities |

---

## 3. Authentication

Authentication is the process of verifying identity — proving "you are who you say you are."

### The Authentication Flow (4 Steps)

**Step 1** — Application collects credentials and creates a token:
```java
UsernamePasswordToken token = new UsernamePasswordToken("jsmith", "mypassword");
token.setRememberMe(true); // optional
```

**Step 2** — Subject submits the token:
```java
Subject currentUser = SecurityUtils.getSubject();
try {
    currentUser.login(token);
} catch (UnknownAccountException uae) {
    // username not found
} catch (IncorrectCredentialsException ice) {
    // wrong password
} catch (LockedAccountException lae) {
    // account locked
} catch (AuthenticationException ae) {
    // catch-all
}
```

**Step 3** — `SecurityManager` delegates to `ModularRealmAuthenticator`, which calls configured Realms.

**Step 4** — Realm looks up the user in the data store, verifies credentials, and returns an `AuthenticationInfo` object.

### Multi-Realm Authentication Strategies

| Strategy | Behavior |
|---|---|
| `AtLeastOneSuccessfulStrategy` | (Default) At least one Realm must succeed |
| `FirstSuccessfulStrategy` | First successful Realm wins; others skipped |
| `AllSuccessfulStrategy` | ALL Realms must succeed |

### Logout
```java
currentUser.logout(); // clears session, invalidates auth state
```

---

## 4. Authorization

Authorization answers: *"Is this user allowed to do X?"*

### Three Approaches

#### 1. Programmatic (Role-Based)
```java
Subject currentUser = SecurityUtils.getSubject();

if (currentUser.hasRole("administrator")) {
    // show admin UI
}

// Assertion style (throws AuthorizationException if fails)
currentUser.checkRole("bankTeller");
openBankAccount();
```

#### 2. Programmatic (Permission-Based)
```java
// Boolean check
if (currentUser.isPermitted("printer:print:laserjet4400n")) {
    // allow print
}

// Assertion style
currentUser.checkPermission("account:open");
openBankAccount();
```

#### 3. Annotation-Based (AOP required)
```java
@RequiresRoles("administrator")
public void deleteUser(User user) { ... }

@RequiresPermissions("account:create")
public void createAccount(Account account) { ... }

@RequiresAuthentication
public void updateAccount(Account account) { ... }

@RequiresGuest
public void signUp(User newUser) { ... }

@RequiresUser  // authenticated OR remembered
public void viewProfile() { ... }
```

### Authorization Sequence
1. App calls `subject.isPermitted("x:y:z")`
2. Subject delegates to `SecurityManager`
3. SecurityManager delegates to `ModularRealmAuthorizer`
4. Authorizer iterates over Realms calling `isPermitted()` on each
5. First `true` result short-circuits; returns immediately

---

## 5. Permissions (WildcardPermission System)

This is Shiro's most powerful feature. Permissions define raw capabilities — the *what*, not the *who*.

### Permission Philosophy

> A permission is "a statement that defines an explicit behavior or action." It says what *can be done*, never *who* can do it.

### WildcardPermission Syntax: `domain:action:instance`

Parts are separated by `:`. Each part can use wildcards (`*`) and comma-separated multiple values.

#### Simple Permission
```
queryPrinter          → can query printers (flat string)
```

#### Domain:Action (Two-Part)
```
printer:query         → can query printers
printer:print         → can print
printer:manage        → can manage printers
```

#### Multiple Actions (Comma Separated)
```
printer:print,query   → can both print AND query
```

#### Wildcard Action (All Actions)
```
printer:*             → can do anything with printers
```

#### All Domains
```
*:view                → can view everything in every domain
```

#### Instance-Level (Three-Part)
```
printer:query:lp7200        → can query printer lp7200 specifically
printer:print:epsoncolor    → can print to epsoncolor only
printer:*:lp7200            → can do anything to lp7200
printer:print:*             → can print to any printer
printer:*:*                 → full printer access
```

### Missing Parts (Implied Wildcards)

Omitting trailing parts implies `*`:
```
printer              ≡ printer:*:*
printer:print        ≡ printer:print:*
```

> ⚠️ You can ONLY omit parts from the END, not the middle.
> `printer:lp7200` is NOT equivalent to `printer:*:lp7200`

### Implication vs. Equality

Permission checks use **implication logic**, not string equality.

```
user:*    implies    user:delete    (true)
user:*    implies    user:view      (true)
user:*:12345 implies user:update:12345  (true)
```

This is why you should:
- **Assign** broad permissions: `printer:*`
- **Check** specific permissions at runtime: `isPermitted("printer:print:lp7200")`

### Runtime Check Best Practice
```java
// CORRECT — specific check
if (SecurityUtils.getSubject().isPermitted("printer:print:lp7200")) {
    printDocument(doc, "lp7200");
}

// INCORRECT — too broad; blocks users who CAN print to lp7200
// but don't have blanket printer:print access
if (SecurityUtils.getSubject().isPermitted("printer:print")) {
    printDocument(doc, "lp7200");
}
```

### Performance Note

Each permission check iterates all assigned permissions for the user. Use a `CacheManager` to cache user/role/permission data and avoid repeated DB hits. Shiro's `AuthorizingRealm` supports caching natively.

---

## 6. Realms

A Realm is the bridge between Shiro and your security data store.

### Built-in Realms

| Realm | Data Source |
|---|---|
| `IniRealm` | shiro.ini file (dev/testing) |
| `JdbcRealm` | Any JDBC-compatible database |
| `JndiLdapRealm` | LDAP / Active Directory |
| `ActiveDirectoryRealm` | Microsoft Active Directory |
| `PropertiesRealm` | Java `.properties` file |

### Custom Realm (most common pattern in OFBiz)

```java
public class MyCustomRealm extends AuthorizingRealm {

    // Authentication: verify username/password
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(
            AuthenticationToken token) throws AuthenticationException {

        UsernamePasswordToken upToken = (UsernamePasswordToken) token;
        String username = upToken.getUsername();

        // Look up user in your DB
        User user = userDao.findByUsername(username);
        if (user == null) throw new UnknownAccountException();

        return new SimpleAuthenticationInfo(
            user.getUsername(),
            user.getHashedPassword(),
            ByteSource.Util.bytes(user.getSalt()),
            getName()
        );
    }

    // Authorization: load roles/permissions for user
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(
            PrincipalCollection principals) {

        String username = (String) principals.getPrimaryPrincipal();
        User user = userDao.findByUsername(username);

        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
        info.setRoles(user.getRoles());           // Set<String>
        info.setStringPermissions(user.getPermissions()); // Set<String>
        return info;
    }
}
```

### Credential Matching (Hashed Passwords)

```java
// In realm configuration
HashedCredentialsMatcher matcher = new HashedCredentialsMatcher();
matcher.setHashAlgorithmName("SHA-256");
matcher.setHashIterations(1024);
matcher.setStoredCredentialsHexEncoded(false); // use Base64

realm.setCredentialsMatcher(matcher);
```

---

## 7. Session Management

Shiro provides its own session layer — independent of the Servlet container. This means sessions work in CLI apps, REST services, and web apps uniformly.

```java
Subject currentUser = SecurityUtils.getSubject();
Session session = currentUser.getSession();

// Store/retrieve data
session.setAttribute("key", someValue);
Object value = session.getAttribute("key");

// Session timeout
session.setTimeout(1800000); // 30 minutes in ms

// Invalidate
session.stop();
```

### Session Clustering
Shiro sessions can be stored in any backend (Hazelcast, Redis, etc.) by implementing `SessionDAO`.

---

## 8. Cryptography

Shiro wraps Java's complex JCE with a simple API.

### Password Hashing
```java
// Hash a password with salt
String plaintext = "mypassword";
String hashed = new Sha256Hash(plaintext, salt, 1024).toBase64();

// Using the more flexible HashService
DefaultHashService hashService = new DefaultHashService();
hashService.setHashAlgorithmName("SHA-256");
hashService.setHashIterations(500000);
hashService.setGenerateSalt(true);

HashRequest request = new HashRequest.Builder()
    .setAlgorithmName("SHA-256")
    .setSource(ByteSource.Util.bytes("mypassword"))
    .build();
Hash result = hashService.computeHash(request);
```

### Symmetric Encryption
```java
AesCipherService cipherService = new AesCipherService();
cipherService.setKeySize(128);

Key key = cipherService.generateNewKey();
byte[] encrypted = cipherService.encrypt("Hello".getBytes(), key.getEncoded()).getBytes();
byte[] decrypted = cipherService.decrypt(encrypted, key.getEncoded()).getBytes();
```

---

## 9. Web Integration

### Setup: web.xml

```xml
<!-- Shiro listener -->
<listener>
    <listener-class>org.apache.shiro.web.env.EnvironmentLoaderListener</listener-class>
</listener>

<!-- Shiro filter — must be first filter -->
<filter>
    <filter-name>ShiroFilter</filter-name>
    <filter-class>org.apache.shiro.web.servlet.ShiroFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>ShiroFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

### URL-Based Access Control (shiro.ini)

```ini
[urls]
/login.html         = anon
/logout             = logout
/api/**             = authcBasic
/admin/**           = authc, roles[admin]
/account/**         = authc, perms[account:view]
/**                 = authc
```

### Built-in Filters

| Filter | Description |
|---|---|
| `anon` | Anonymous access — no login required |
| `authc` | Form-based authentication required |
| `authcBasic` | HTTP Basic authentication required |
| `logout` | Handles logout and redirect |
| `roles[role1,role2]` | Requires the specified roles |
| `perms[perm1,perm2]` | Requires the specified permissions |
| `user` | Authenticated OR RememberMe |
| `ssl` | HTTPS required |
| `noSessionCreation` | Prevents session creation |

### JSP Tag Library

```jsp
<%@ taglib prefix="shiro" uri="http://shiro.apache.org/tags" %>

<shiro:authenticated>  <!-- Only shown when logged in -->
    <p>Welcome, <shiro:principal/>!</p>
</shiro:authenticated>

<shiro:hasRole name="administrator">
    <a href="/admin">Admin Panel</a>
</shiro:hasRole>

<shiro:hasPermission name="printer:print">
    <button>Print</button>
</shiro:hasPermission>

<shiro:guest>
    <a href="/login">Login</a>
</shiro:guest>
```

---

## 10. Annotations & Declarative Security

Requires AOP (Spring AOP, AspectJ, or Guice AOP).

| Annotation | Purpose |
|---|---|
| `@RequiresAuthentication` | User must be logged in this session |
| `@RequiresUser` | User logged in OR remembered |
| `@RequiresGuest` | Must be anonymous (not logged in) |
| `@RequiresRoles("admin")` | Must have the specified role(s) |
| `@RequiresPermissions("x:y")` | Must have the specified permission(s) |

---

## 11. Configuration (shiro.ini)

```ini
[main]
# Define Realm
myRealm = com.example.MyJdbcRealm
myRealm.dataSource = $dataSource

# Credential Matcher
credentialsMatcher = org.apache.shiro.authc.credential.HashedCredentialsMatcher
credentialsMatcher.hashAlgorithmName = SHA-256
credentialsMatcher.hashIterations = 1024
myRealm.credentialsMatcher = $credentialsMatcher

# CacheManager
cacheManager = org.apache.shiro.cache.ehcache.EhCacheManager
securityManager.cacheManager = $cacheManager

# Assign realm
securityManager.realms = $myRealm

[users]
# Format: username = password, role1, role2
admin = $2a$secret_hash, admin, user
jsmith = $2a$other_hash, user

[roles]
# Format: rolename = permission1, permission2
admin = *
user = account:view, profile:edit

[urls]
/login  = anon
/logout = logout
/**     = authc
```

---

## 12. Apache OFBiz + Shiro Integration

### Overview

OFBiz's application framework uses Apache Shiro for authentication and authorization. The configuration file allows a user to select and configure a Realm. In addition to Shiro's built-in Realm choices, the Realm choices include "Native" — a custom Realm implementation that uses OFBiz's entity engine for persistence.

### Design Goals for OFBiz+Shiro

The design goals are: easy integration with existing authentication and authorization infrastructure; leverage the external library with very little custom code; and thread-safety.

The architecture uses:
- **Library:** Apache Shiro + OFBiz extensions
- **Java package:** `org.apache.ofbiz.foundation.security`
- The "Native" Realm translates OFBiz's `SecurityGroup` / `SecurityPermission` entity model into Shiro's `AuthorizationInfo`

### How OFBiz Extends Shiro

OFBiz wraps Shiro via the `org.ofbiz.security.Security` interface, which provides:

```java
// Core OFBiz security API (backed by Shiro underneath)
security.hasPermission(permission, userLogin)
security.hasEntityPermission(entity, action, userLogin)
security.hasRolePermission(entity, action, primaryKey, role, userLogin)
```

---

## 13. OFBiz Security Permission Model (Deep Dive)

### Two Categories of Security

Security in OFBiz is split into 2 categories:

**Category 1 (user permission, e.g. `ORDERMGR_CREATE`)** is UserLogin-dependent and doesn't know about anything except the UserLogin, the permissions checked for different screens/services, and the SecurityGroup structure that maps between them.

**Category 2 (party permission, e.g. `ORDERMGR_ROLE_CREATE`)** is Party-dependent and can be combined with Category 1, usually with special "role limited" permissions that require not just the permission, but some relationship between the Party and whatever artifact is being controlled.

### Permission Naming Convention

Category 1 is based on **SecurityPermissions** records. The names of permissions consist of two parts separated by a `_` character. Most of the time, the first part is the application name and the second part is the action or operation. The application name is normally the same as the component name written in uppercase. The action can be anything; however VIEW, CREATE, UPDATE, DELETE (CRUD), and ADMIN are most commonly used. ADMIN is a special action that allows all operations.

**Examples:**
```
ORDERMGR_VIEW       → View orders
ORDERMGR_CREATE     → Create orders
ORDERMGR_UPDATE     → Update orders
ORDERMGR_DELETE     → Delete orders
ORDERMGR_ADMIN      → All order operations

CATALOG_VIEW
CATALOG_CREATE
CATALOG_ROLE_CREATE → Role-limited catalog create
```

### Two Enforcement Patterns

OFBiz follows two general patterns for enforcing security permissions:

- **The "Application Pattern"** — grants user permissions for a particular application area.
- **The "Role Limited Pattern"** — grants user permissions based upon the user's association with a particular element.

For example: a user is assigned the `ORDERMGR_VIEW` permission and is associated with a particular facility (e.g. XYZ Company) with the `ORDERMGR_ROLE_UPDATE` security role. This combination would allow the user to view orders for ALL facilities, but update orders for XYZ Company only.

---

## 14. OFBiz Permission Enforcement Points

OFBiz enforces permissions at **seven distinct levels**:

### Level 1: Login / Component Access
The **base-permission** defined in `ofbiz-component.xml` as an attribute of the `<webapp>` element controls access to the component. To log in, a user must have at least the `COMPONENT-NAME_VIEW` or `COMPONENT-NAME_ADMIN` permission. Most components require both `OFBTOOLS` AND a component-specific permission (comma = AND operator).

```xml
<!-- ofbiz-component.xml -->
<webapp name="ordermgr"
        base-permission="OFBTOOLS,ORDERMGR"
        .../>
```

### Level 2: Component Menu
The component top-level menu will only show components to which a logged-on user has at least the `WEBAPP-NAME_VIEW` or `COMPONENT-NAME_ADMIN` permission. This is done in `appbar.ftl`.

### Level 3: Request Level (controller.xml)
```xml
<!-- controller.xml -->
<request-map uri="createOrder">
    <security https="true" auth="true"/>
    <event type="service" invoke="createOrder"/>
    <response name="success" type="view" value="orderView"/>
</request-map>
```

### Level 4: Screen Level
```xml
<!-- In screen definition XML -->
<section>
    <condition>
        <if-has-permission permission="ORDERMGR" action="_CREATE"/>
    </condition>
    <widgets>
        <include-form name="CreateOrder" .../>
    </widgets>
    <fail-widgets>
        <label text="Permission denied"/>
    </fail-widgets>
</section>
```

### Level 5: Freemarker Template Level
```ftl
${#-- In .ftl template files --}
<#if security.hasEntityPermission("ORDERMGR", "_CREATE", userLogin)>
    <button>Create Order</button>
</#if>
```

### Level 6: Service Definition Level
```xml
<!-- servicedef/services.xml -->
<service name="createOrder" engine="java" ...>
    <permission-service service-name="orderGenericPermission"
                        main-action="CREATE"/>
    <attribute name="orderId" .../>
</service>
```

### Level 7: Service Implementation (Java/Groovy)
```java
// Java implementation
public static Map<String, Object> createOrder(
        DispatchContext dctx, Map<String, Object> context) {

    Security security = dctx.getSecurity();
    GenericValue userLogin = (GenericValue) context.get("userLogin");

    if (!security.hasEntityPermission("ORDERMGR", "_CREATE", userLogin)) {
        return ServiceUtil.returnError("Permission denied: ORDERMGR_CREATE required");
    }
    // proceed with order creation...
}
```

```groovy
// Groovy service
if (!security.hasEntityPermission("ORDERMGR", "_CREATE", parameters.userLogin)) {
    return error("You do not have permission to create orders.")
}
```

---

## 15. OFBiz Data Model for Security

All security levels rely on several database tables:

1. **SecurityPermission** — lowest level; all basic permissions defined here (e.g. `CATALOG_ADMIN`, `CATALOG_CREATE`, `CATALOG_ROLE_CREATE`).
2. **SecurityGroup** — permission groups (e.g. `FULLADMIN`, `PARTYADMIN`).
3. **SecurityGroupPermission** — associates `SecurityPermission` with `SecurityGroup`.
4. **UserLoginSecurityGroup** — which UserLogin IDs have access to which permission groups.
5. **PartyRelationship** — security groups can be set for certain party relationships.
6. **\*Role entities** (e.g. `OrderRole`, `PartyRole`, `ProductStoreRole`) — used for record-level, role-limited permissions.

### Entity Relationship Diagram (Text)

```
UserLogin ──── UserLoginSecurityGroup ──── SecurityGroup
                                               |
                                    SecurityGroupPermission
                                               |
                                        SecurityPermission

UserLogin ──── Party ──── PartyRole ──── RoleType
                   |
              OrderRole, ProductStoreRole, ContentRole, etc.
              (used for role-limited/record-level permissions)
```

---

## 16. Code Examples: Shiro in OFBiz

### OFBiz Custom Realm (Native Realm Pattern)

```java
public class OFBizRealm extends AuthorizingRealm {

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(
            AuthenticationToken token) throws AuthenticationException {

        UsernamePasswordToken upToken = (UsernamePasswordToken) token;
        String username = upToken.getUsername();

        // OFBiz entity engine lookup
        try {
            GenericValue userLogin = EntityQuery.use(delegator)
                .from("UserLogin")
                .where("userLoginId", username)
                .queryOne();

            if (userLogin == null) throw new UnknownAccountException();
            if ("Y".equals(userLogin.getString("isSystem"))) {
                throw new UnknownAccountException();
            }

            String currentPassword = userLogin.getString("currentPassword");
            return new SimpleAuthenticationInfo(username, currentPassword, getName());

        } catch (GenericEntityException e) {
            throw new AuthenticationException(e);
        }
    }

    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(
            PrincipalCollection principals) {

        String username = (String) principals.getPrimaryPrincipal();
        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();

        try {
            // Load security groups for this user
            List<GenericValue> groups = EntityQuery.use(delegator)
                .from("UserLoginSecurityGroup")
                .where("userLoginId", username)
                .filterByDate()
                .queryList();

            for (GenericValue group : groups) {
                String groupId = group.getString("groupId");
                info.addRole(groupId);

                // Load permissions for this group
                List<GenericValue> groupPerms = EntityQuery.use(delegator)
                    .from("SecurityGroupPermission")
                    .where("groupId", groupId)
                    .filterByDate()
                    .queryList();

                for (GenericValue perm : groupPerms) {
                    info.addStringPermission(perm.getString("permissionId"));
                }
            }
        } catch (GenericEntityException e) {
            throw new AuthorizationException(e);
        }
        return info;
    }
}
```

### Role-Limited Permission Check (Groovy, OFBiz Pattern)

```groovy
/**
 * Check Product-Related Permission — OFBiz canonical pattern
 */
Map checkProductRelatedPermission(String callingMethodName, String checkAction) {
    callingMethodName = callingMethodName ?:
        UtilProperties.getMessage('CommonUiLabels', 'CommonPermissionThisOperation', parameters.locale)

    if (UtilValidate.isEmpty(checkAction)) {
        checkAction = 'UPDATE'
    }

    List roleCategories = []

    // Find all role-categories this product belongs to for this user
    if (parameters.productId &&
            !security.hasEntityPermission('CATALOG', "_${checkAction}", parameters.userLogin)) {

        roleCategories = from('ProductCategoryMemberAndRole')
            .where([productId: parameters.productId,
                    partyId: userLogin.partyId,
                    roleTypeId: 'LTD_ADMIN'])
            .filterByDate('roleFromDate', 'roleThruDate')
            .queryList()
    }

    // Three ways to be authorized:
    // 1. Direct CATALOG permission
    // 2. Role-limited CATALOG_ROLE permission + matching role category
    // 3. Alternate permission root
    if (!(security.hasEntityPermission('CATALOG', "_${checkAction}", parameters.userLogin)
            || (roleCategories &&
                security.hasEntityPermission('CATALOG_ROLE', "_${checkAction}", parameters.userLogin))
            || (parameters.alternatePermissionRoot &&
                security.hasEntityPermission(parameters.alternatePermissionRoot,
                    "_${checkAction}", parameters.userLogin)))) {

        return error(UtilProperties.getMessage('ProductUiLabels',
            "ProductCatalog${checkAction}PermissionError", parameters.locale))
    }
    return success()
}
```

### Protected View Configuration

```xml
<!-- Define a protected view to prevent brute-force/scraping -->
<!-- In Security Admin UI or seed data: -->
<SecurityGroupView
    groupId="PUBLIC_USERS"
    viewNameId="/catalog/control/ViewProduct"
    maxHits="10"
    hitBufferSize="600"
    blockTime="60"/>
```

---

## 17. Best Practices & Tips

### Shiro Best Practices

1. **Always use specific permission checks at runtime** — check `printer:print:lp7200`, not `printer:print`.
2. **Use explicit roles with permissions, not implicit roles** — avoid relying on role *names* alone to imply behavior.
3. **Cache aggressively** — configure a `CacheManager`; auth lookups are expensive.
4. **Hash passwords with salt + iterations** — use `HashedCredentialsMatcher` with SHA-256 and ≥ 1024 iterations (prefer Argon2 or bcrypt for new systems).
5. **Enable RememberMe carefully** — use it for low-risk operations; require full re-authentication for sensitive ones.
6. **Use `checkPermission()` over `isPermitted()` where cleanliness matters** — throws exception automatically, avoids if/else boilerplate.

### OFBiz-Specific Tips

1. **Use `hasEntityPermission()` over `hasPermission()`** — it handles the `_ADMIN` permission automatically (admin implies all actions for that entity).
2. **Prefer permission services over inline checks** — putting security logic in `<permission-service>` allows ECA-based extension without code changes.
3. **Understand the `OFBTOOLS` requirement** — most OFBiz webapps require BOTH `OFBTOOLS` AND the component-specific permission for login.
4. **Use SecurityGroup batching** — assign permissions to groups, groups to users; never assign permissions one-by-one to individual users.
5. **Test role-limited permissions carefully** — a role-limited permission (`ORDERMGR_ROLE_UPDATE`) without a corresponding enforced relationship is effectively a no-op and may silently pass or fail.
6. **Use Protected Views for rate limiting** — configure sensitive views with hit limits and block times to prevent scraping/brute-force.
7. **Migrate from `base-permission` to `access-permission`** — `base-permission` is deprecated; plan migration when upgrading.

---

## Summary Comparison: Shiro Concepts → OFBiz Equivalents

| Apache Shiro | Apache OFBiz |
|---|---|
| `Subject` | `UserLogin` entity |
| `Realm` | OFBiz Native Realm (entity engine) |
| `SecurityManager` | `Security` service / `OFBizSecurity` class |
| `Permission` string | `SecurityPermission.permissionId` (e.g. `ORDERMGR_CREATE`) |
| `Role` | `SecurityGroup` entity |
| Role assignment | `UserLoginSecurityGroup` entity |
| Permission assignment | `SecurityGroupPermission` entity |
| Instance-level permission | `OrderRole`, `PartyRole`, `ProductStoreRole` entities |
| `@RequiresPermissions` | `<if-has-permission>` tag in screen XML |
| `isPermitted()` | `security.hasEntityPermission()` in Java/Groovy |
| URL filter chain | `controller.xml` request-map + `<security auth="true"/>` |
| Session | OFBiz `HttpSession` + Shiro `SessionManager` |

---

*Documentation compiled from: [shiro.apache.org](https://shiro.apache.org), [shiro.apache.org/permissions.html](https://shiro.apache.org/permissions.html), [shiro.apache.org/authorization.html](https://shiro.apache.org/authorization.html), and [cwiki.apache.org/confluence/display/OFBIZ](https://cwiki.apache.org/confluence/display/OFBIZ/OFBiz+Security+Permissions)*