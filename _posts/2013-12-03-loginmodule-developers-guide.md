---
layout: post
category : token-auth
tagline: "Supporting tagline"
tags : [JAAS, Security]
---
{% include JB/setup %}

<script src="http://sdp-dev.sh.intel.com:4000/assets/js/jquery.toc.js"></script>


# Overview
The JavaTM Authentication and Authorization Service (JAAS) was introduced as an optional package to the JavaTM 2 SDK, Standard Edition (J2SDK), v 1.3. JAAS was integrated into the Java TM Standard Edition Development Kit starting with J2SDK 1.4.

JAAS provides subject-based authorization on authenticated identities. This document focuses on the authentication aspect of JAAS, specifically the LoginModule interface.

Who Should Read This Document
This document is intended for experienced programmers who require the ability to write a LoginModule implementing an authentication technology.

Related Documentation
This document assumes you have already read the following:

JAAS Reference Guide
It also discusses various classes and interfaces in the JAAS API. Please reference the javadocs for the JAAS API specification for more detailed information:

javax.security.auth Package
javax.security.auth.callback Package
javax.security.auth.kerberos Package
javax.security.auth.login Package
javax.security.auth.spi Package
javax.security.auth.x500 Package
com.sun.security.auth Package
com.sun.security.auth.callback Package
com.sun.security.auth.login Package
com.sun.security.auth.module Package
The following tutorials for JAAS authentication and authorization can be run by everyone:

JAAS Authentication Tutorial
JAAS Authorization Tutorial
Similar tutorials for JAAS authentication and authorization, but which demonstrate the use of a Kerberos LoginModule and thus which require a Kerberos installation, can be found at

JAAS Authentication
JAAS Authorization
These two tutorials are a part of the Java GSS-API and JAAS sequence of tutorials that utilize Kerberos as the underlying technology for authentication and secure communication.

Introduction
The LoginModule documentation describes the interface that must be implemented by authentication technology providers. LoginModules are plugged in under applications to provide a particular type of authentication.

While applications write to the LoginContext Application Programming Interface (API), authentication technology providers implement the LoginModule interface. A Configuration specifies the LoginModule(s) to be used with a particular login application. Different LoginModules can be plugged in under the application without requiring any modifications to the application itself.

The LoginContext is responsible for reading the Configuration and instantiating the specified LoginModules. Each LoginModule is initialized with a Subject, a CallbackHandler, shared LoginModule state, and LoginModule-specific options.

The Subject represents the user or service currently being authenticated and is updated by a LoginModule with relevant Principals and credentials if authentication succeeds. LoginModules use the CallbackHandler to communicate with users (to prompt for user names and passwords, for example), as described in the login method description. Note that the CallbackHandler may be null. A LoginModule that requires a CallbackHandler to authenticate the Subject may throw a LoginException if it was initialized with a null CallbackHandler. LoginModules optionally use the shared state to share information or data among themselves.

The LoginModule-specific options represent the options configured for this LoginModule in the login Configuration. The options are defined by the LoginModule itself and control the behavior within it. For example, a LoginModule may define options to support debugging/testing capabilities. Options are defined using a key-value syntax, such as debug=true. The LoginModule stores the options as a Map so that the values may be retrieved using the key. Note that there is no limit to the number of options a LoginModule chooses to define.

The calling application sees the authentication process as a single operation invoked via a call to the LoginContext's login method. However, the authentication process within each LoginModule proceeds in two distinct phases. In the first phase of authentication, the LoginContext's login method invokes the login method of each LoginModule specified in the Configuration. The login method for a LoginModule performs the actual authentication (prompting for and verifying a password for example) and saves its authentication status as private state information. Once finished, the LoginModule's login method returns true (if it succeeded) or false (if it should be ignored), or it throws a LoginException to specify a failure. In the failure case, the LoginModule must not retry the authentication or introduce delays. The responsibility of such tasks belongs to the application. If the application attempts to retry the authentication, each LoginModule's login method will be called again.

In the second phase, if the LoginContext's overall authentication succeeded (calls to the relevant required, requisite, sufficient and optional LoginModules' login methods succeeded), then the commit method for each LoginModule gets invoked. (For an explanation of the LoginModule flags required, requisite, sufficient and optional, please consult the javax.security.auth.login.Configuration documentation and Appendix B: Login Configuration Files in the JAAS Reference Guide.) The commit method for a LoginModule checks its privately saved state to see if its own authentication succeeded. If the overall LoginContext authentication succeeded and the LoginModule's own authentication succeeded, then the commit method associates the relevant Principals (authenticated identities) and credentials (authentication data such as cryptographic keys) with the Subject.

If the LoginContext's overall authentication failed (the relevant REQUIRED, REQUISITE, SUFFICIENT and OPTIONAL LoginModules' login methods did not succeed), then the abort method for each LoginModule gets invoked. In this case, the LoginModule removes/destroys any authentication state originally saved.

Logging out a Subject involves only one phase. The LoginContext invokes the LoginModule's logout method. The logout method for the LoginModule then performs the logout procedures, such as removing Principals or credentials from the Subject, or logging session information.

Steps to Implement a LoginModule
The steps required in order to implement and test a LoginModule are the following:

Step 1: Understand the Authentication Technology
Step 2: Name the LoginModule Implementation
Step 3: Implement the Abstract LoginModule Methods
Step 4: Choose or Write a Sample Application
Step 5: Compile the LoginModule and Application
Step 6: Prepare for Testing
Step 6a: Place Your LoginModule and Application Code in JAR Files
Step 6b: Decide Where to Store the JAR Files
Step 6c: Set LoginModule and Application JAR File Permissions
Step 6d: Create a Configuration Referencing the LoginModule
Step 7: Test Use of the LoginModule
Step 8: Document Your LoginModule Implementation
Step 9: Make LoginModule JAR File and Documents Available
The steps required to implement and test a new LoginModule follow. Please reference the SampleLoginModule and other files described in the JAAS Reference Guide for examples of what may be done for the various steps.

Step 1: Understand the Authentication Technology
The first thing you need to do is understand the authentication technology to be implemented by your new LoginModule provider, and determine its requirements.
One thing you will need to determine is whether or not your LoginModule will require some form of user interaction (retrieving a user name and password, for example). If so, you will need to become familiar with the CallbackHandler interface and the javax.security.auth.callback package. In that package you will find several possible Callback implementations to use. (Alternatively, you can create your own Callback implementations.) The LoginModule will invoke the CallbackHandler specified by the application itself and passed to the LoginModule's initialize method. The LoginModule passes the CallbackHandler an array of appropriate Callbacks. See the login method in Step 3.

Note that it is possible for LoginModule implementations not to have any end-user interactions. Such LoginModules would not need to access the callback package.

Another thing you should determine is what configuration options you want to make available to the user, who specifies configuration information in whatever form the current Configuration implementation expects (for example, in files). For each option, decide the option name and possible values. For example, if a LoginModule may be configured to consult a particular authentication server host, decide on the option's key name ("auth_server", for example), as well as the possible server hostnames valid for that option ("server_one.foo.com" and "server_two.foo.com", for example).

Step 2: Name the LoginModule Implementation
Decide on the proper package and class name for your LoginModule.
For example, a LoginModule developed by IBM might be called com.ibm.auth.Module where com.ibm.auth is the package name and Module is the name of the LoginModule class implementation.

Step 3: Implement the Abstract LoginModule Methods
The LoginModule interface specifies five abstract methods that require implementations:

initialize
login
commit
abort
logout
See below and the LoginModule API for more information on each method above.
In addition to these methods, a LoginModule implementation must provide a public constructor with no arguments. This allows for its proper instantiation by a LoginContext. Note that if no such constructor is provided in your LoginModule implementation, a default no-argument constructor is automatically inherited from the Object class.

public void initialize (Subject subject,
CallbackHandler handler,
Map<java.lang.String, ?> sharedState,
Map<java.lang.String, ?> options) { ... }
The initialize method is called to initialize the LoginModule with the relevant authentication and state information.

This method is called by a LoginContext immediately after this LoginModule has been instantiated, and prior to any calls to its other public methods. The method implementation should store away the provided arguments for future use.

The initialize method may additionally peruse the provided sharedState to determine what additional authentication state it was provided by other LoginModules, and may also traverse through the provided options to determine what configuration options were specified to affect the LoginModule's behavior. It may save option values in variables for future use.

Note: JAAS LoginModules may use the options defined in PAM (use_first_pass, try_first_pass, use_mapped_pass, and try_mapped_pass) to achieve single-signon. See Making Login Services Independent from Authentication Technologies for further information.

Below is a list of options commonly supported by LoginModules. Note that the following is simply a guideline. Modules are free to support a subset (or none) of the following options.

try_first_pass - If true, the first LoginModule in the stack saves the password entered, and subsequent LoginModules also try to use it. If authentication fails, the LoginModules prompt for a new password and retry the authentication.
use_first_pass - If true, the first LoginModule in the stack saves the password entered, and subsequent LoginModules also try to use it. LoginModules do not prompt for a new password if authentication fails (authentication simply fails).
try_mapped_pass - If true, the first LoginModule in the stack saves the password entered, and subsequent LoginModules attempt to map it into their service-specific password. If authentication fails, the LoginModules prompt for a new password and retry the authentication.
use_mapped_pass - If true, the first LoginModule in the stack saves the password entered, and subsequent LoginModules attempt to map it into their service-specific password. LoginModules do not prompt for a new password if authentication fails (authentication simply fails).
moduleBanner - If true, then when invoking the CallbackHandler, the LoginModule provides a TextOutputCallback as the first Callback, which describes the LoginModule performing the authentication.
debug - If true, instructs a LoginModule to output debugging information.
The initialize method may freely ignore state or options it does not understand, although it would be wise to log such an event if it does occur.

Note that the LoginContext invoking this LoginModule (and the other configured LoginModules, as well), all share the same references to the provided Subject and sharedState. Modifications to the Subject and sharedState will, therefore, be seen by all.

boolean login() throws LoginException;
The login method is called to authenticate a Subject. This is phase 1 of authentication.

This method implementation should perform the actual authentication. For example, it may cause prompting for a user name and password, and then attempt to verify the password against a password database. Another example implementation may inform the user to insert their finger into a fingerprint reader, and then match the input fingerprint against a fingerprint database.

If your LoginModule requires some form of user interaction (retrieving a user name and password, for example), it should not do so directly. That is because there are various ways of communicating with a user, and it is desirable for LoginModules to remain independent of the different types of user interaction. Rather, the LoginModule's login method should invoke the handle method of the the CallbackHandler passed to the initialize method to perform the user interaction and set appropriate results, such as the user name and password. The LoginModule passes the CallbackHandler an array of appropriate Callbacks, for example a NameCallback for the user name and a PasswordCallback for the password, and the CallbackHandler performs the requested user interaction and sets appropriate values in the Callbacks. For example, to process a NameCallback, the CallbackHandler may prompt for a name, retrieve the value from the user, and call the NameCallback's setName method to store the name.

The authentication process may also involve communication over a network. For example, if this method implementation performs the equivalent of a kinit in Kerberos, then it would need to contact the KDC. If a password database entry itself resides in a remote naming service, then that naming service needs to be contacted, perhaps via the Java Naming and Directory Interface (JNDI). Implementations might also interact with an underlying operating system. For example, if a user has already logged into an operating system like Solaris or Windows NT, this method might simply import the underlying operating system's identity information.

The login method should

Determine whether or not this LoginModule should be ignored. One example of when it should be ignored is when a user attempts to authenticate under an identity irrelevant to this LoginModule (if a user attempts to authenticate as root using NIS, for example). If this LoginModule should be ignored, login should return false. Otherwise, it should do the following:
Call the CallbackHandler handle method if user interaction is required.
Perform the authentication.
Store the authentication result (success or failure).
If authentication succeeded, save any relevant state information that may be needed by the commit method.
Return true if authentication succeeds, or throw a LoginException such as FailedLoginException if authentication fails.
Note that the login method implementation should not associate any new Principal or credential information with the saved Subject object. This method merely performs the authentication, and then stores away the authentication result and corresponding authentication state. This result and state will later be accessed by the commit or abort method. Note that the result and state should typically not be saved in the sharedState Map, as they are not intended to be shared with other LoginModules.

An example of where this method might find it useful to store state information in the sharedState Map is when LoginModules are configured to share passwords. In this case, the entered password would be saved as shared state. By sharing passwords, the user only enters the password once, and can still be authenticated to multiple LoginModules. The standard conventions for saving and retrieving names and passwords from the sharedState Map are the following:

javax.security.auth.login.name - Use this as the shared state map key for saving/retrieving a name.
javax.security.auth.login.password - Use this as the shared state map key for saving/retrieving a password.
If authentication fails, the login method should not retry the authentication. This is the responsibility of the application. Multiple LoginContext login method calls by an application are preferred over multiple login attempts from within LoginModule.login().

boolean commit() throws LoginException;
The commit method is called to commit the authentication process. This is phase 2 of authentication when phase 1 succeeds. It is called if the LoginContext's overall authentication succeeded (that is, if the relevant REQUIRED, REQUISITE, SUFFICIENT and OPTIONAL LoginModules succeeded.)

This method should access the authentication result and corresponding authentication state saved by the login method.

If the authentication result denotes that the login method failed, then this commit method should remove/destroy any corresponding state that was originally saved.

If the saved result instead denotes that this LoginModule's login method succeeded, then the corresponding state information should be accessed to build any relevant Principal and credential information. Such Principals and credentials should then be added to the Subject stored away by the initialize method.

After adding Principals and credentials, dispensable state fields should be destroyed expeditiously. Likely fields to destroy would be user names and passwords stored during the authentication process.

The commit method should save private state indicating whether the commit succeeded or failed.

The following chart depicts what a LoginModule's commit method should return. The different boxes represent the different situations that may occur. For example, the top-left corner box depicts what the commit method should return if both the previous call to login succeeded and the commit method itself succeeded.

     \
LOGIN \ COMMIT
       \
        \   SUCCESS          FAILURE
        +--------------+-----------------+
        |              |                 |
SUCCESS | return TRUE  | throw EXCEPTION |
        |              |                 |
        +--------------+-----------------+
        |              |                 |
FAILURE | return FALSE | return FALSE    |
        |              |                 |
        +--------------+-----------------+

boolean abort() throws LoginException;
The abort method is called to abort the authentication process. This is phase 2 of authentication when phase 1 fails. It is called if the LoginContext's overall authentication failed.

This method first accesses this LoginModule's authentication result and corresponding authentication state saved by the login (and possibly commit) methods, and then clears out and destroys the information. Sample state to destroy would be user names and passwords.

If this LoginModule's authentication attempt failed, then there shouldn't be any private state to clean up.

The following charts depict what a LoginModule's abort method should return. This first chart assumes that the previous call to login succeeded. For instance, the top-left corner box depicts what the abort method should return if both the previous call to login and commit succeeded, and the abort method itself also succeeded.

      \
COMMIT \ ABORT (assumes LOGIN method succeeded)
        \
         \   SUCCESS         FAILURE
        +--------------+-----------------+
        |              |                 |
SUCCESS | return TRUE  | throw EXCEPTION |
        |              |                 |
        +--------------+-----------------+
        |              |                 |
FAILURE | return TRUE  | throw EXCEPTION |
        |              |                 |
        +--------------+-----------------+
The second chart depicts what a LoginModule's abort method should return, assuming that the previous call to login failed. For instance, the top-left corner box depicts what the abort method should return if the previous call to login failed, the previous call to commit succeeded, and the abort method itself also succeeded.

      \
COMMIT \ ABORT (assumes LOGIN method failed)
        \
         \   SUCCESS         FAILURE
        +--------------+-----------------+
        |              |                 |
SUCCESS | return FALSE | return FALSE    |
        |              |                 |
        +--------------+-----------------+
        |              |                 |
FAILURE | return FALSE | return FALSE    |
        |              |                 |
        +--------------+-----------------+
boolean logout() throws LoginException;
The logout method is called to log out a Subject.

This method removes Principals, and removes/destroys credentials associated with the Subject during the commit operation. This method should not touch those Principals or credentials previously existing in the Subject, or those added by other LoginModules.

If the Subject has been marked read-only (the Subject's isReadOnly method returns true), then this method should only destroy credentials associated with the Subject during the commit operation (removing the credentials is not possible). If the Subject has been marked as read-only and the credentials associated with the Subject during the commit operation are not destroyable (they do not implement the Destroyable interface), then this method may throw a LoginException.

The logout method should return true if logout succeeds, or otherwise throw a LoginException.

Step 4: Choose or Write a Sample Application
Either choose an existing sample application for your testing, or write a new one. See JAAS Reference Guide for information about application requirements and a sample application you can use for your testing.

Step 5: Compile the LoginModule and Application
Compile your new LoginModule and the application you will use for testing.
Step 6: Prepare for Testing
Step 6a: Place Your LoginModule and Application Code in JAR Files
Place your LoginModule and application code in separate JAR files, in preparation for referencing the JAR files in the policy in Step 6c. Here is a sample command for creating a JAR file:

jar cvf <JAR file name> <list of classes, separated by spaces>
This command creates a JAR file with the specified name containing the specified classes.

For more information on the jar tool, see jar (for Solaris) (for Microsoft Windows).

Step 6b: Decide Where to Store the JAR Files
The application can be stored essentially anywhere you like.

Your LoginModule can also be placed anywhere you (and other clients) like. If the LoginModule is fully trusted, it can be placed in the JRE's lib/ext (standard extension) directory.

You will need to test the LoginModule being located both in the lib/ext directory and elsewhere because in one situation your LoginModule will need to explicitly be granted permissions required for any security-sensitive operations it does, while in the other case such permissions are not needed.

If your LoginModule is placed in the JRE's lib/ext directory, it will be treated as an installed extension and no permissions need to be granted, since the default system policy file grants all permissions to installed extensions.

If your LoginModule is placed anywhere else, the permissions need to be granted, for example by grant statements in a policy file.

Decide where you will store the LoginModule JAR file for testing the case where it is not an installed extension. In the next step, you grant permissions to the JAR file, in the specified location.

Step 6c: Set LoginModule and Application JAR File Permissions
If your LoginModule and/or application performs security-sensitive tasks that will trigger security checks (making network connections, reading or writing files on a local disk, etc), it will need to be granted the required permissions if it is not an installed extension (see Step 6b) and it is run while a security manager is installed.

Since LoginModules usually associate Principals and credentials with an authenticated Subject, some types of permissions a LoginModule will typically require are AuthPermissions with target names "modifyPrincipals", "modifyPublicCredentials", and "modifyPrivateCredentials".

A sample statement granting permissions to a LoginModule whose code is in MyLM.jar appears below. Such a statement could appear in a policy file. In this example, the MyLM.jar file is assumed to be in the /localWork directory.

grant codeBase "file:/localWork/MyLM.jar" {
  permission javax.security.auth.AuthPermission "modifyPrincipals";
  permission javax.security.auth.AuthPermission "modifyPublicCredentials";
  permission javax.security.auth.AuthPermission "modifyPrivateCredentials";
};
Note: Since a LoginModule is always invoked within an AccessController.doPrivileged call, it should not have to call doPrivileged itself. If it does, it may inadvertently open up a security hole. For example, a LoginModule that invokes the application-provided CallbackHandler inside a doPrivileged call opens up a security hole by permitting the application's CallbackHandler to gain access to resources it would otherwise not have been able to access.

Step 6d: Create a Configuration Referencing the LoginModule
Because JAAS supports a pluggable authentication architecture, your new LoginModule can be used without requiring modifications to existing applications. Only the login Configuration needs to be updated in order to indicate use of a new LoginModule.

The default Configuration implementation from Sun Microsystems reads configuration information from configuration files, as described in com.sun.security.auth.login.ConfigFile.html.

Create a configuration file to be used for testing. For example, to configure the previously-mentioned hypothetical IBM LoginModule for an application, the configuration file might look like this:

    AppName {
        com.ibm.auth.Module REQUIRED debug=true;
    };
where AppName should be whatever name the application uses to refer to this entry in the login configuration file. The application specifies this name as the first argument to the LoginContext constructor.
Step 7: Test Use of the LoginModule
Finally, test your application and its use of the LoginModule. When you run the application, specify the login configuration file to be used. For example, suppose your application is named MyApp, it is located in MyApp.jar, and your configuration file is test.conf. You could run the application and specify the configuration file via the following:

java -classpath MyApp.jar
 -Djava.security.auth.login.config=test.conf MyApp
Type all that on one line. Multiple lines are used here for legibility.

To specify a policy file named my.policy and run the application with a security manager installed, do the following:

java -classpath MyApp.jar -Djava.security.manager
 -Djava.security.policy=my.policy
 -Djava.security.auth.login.config=test.conf MyApp
Again, type all that on one line.

You may want to configure the LoginModule with a debug option to help ensure that it is working correctly.

Debug your code and continue testing as needed. If you have problems, review the steps above and ensure they are all completed.

Be sure to vary user input and the LoginModule options specified in the configuration file.

Be sure to also include testing using different installation options (e.g., making the LoginModule an installed extension or placing it on the class path) and execution environments (with or without a security manager running). Installation options are discussed in Step 6b. In particular, in order to ensure your LoginModule works when a security manager is installed and the LoginModule and application are not installed extensions, you need to test such an installation and execution environment, after granting required permissions, as described in Step 6c.

If you find during testing that your LoginModule or application needs modifications, make the modifications, recompile (Step 5), place the updated code in a JAR file (Step 6a), re-install the JAR file (Step 6b), if needed fix or add to the permissions (Step 6c), if needed modify the login configuration file (Step 6d), and then re-run the application and repeat these steps as needed.

Step 8: Document Your LoginModule Implementation
The next step is to write documentation for clients of your LoginModule. Example documentation you may want to include is:

A README or User Guide describing
the authentication process employed by your LoginModule implementation.
information on how to install the LoginModule.
configuration options accepted by the LoginModule. For each option, specify the option name and possible values (or types of values), as well as the behavior the option controls.
the permissions required by your LoginModule when it is run with a security manager (and it is not an installed extension).
An example Configuration file that references your new LoginModule.
An example policy file granting your LoginModule the required permissions.
API documentation. Putting javadoc comments into your source code as you write it will make the API javadocs easy to generate.
Step 9: Make LoginModule JAR File and Documents Available
The final step is to make your LoginModule JAR file and documentation available to clients.

