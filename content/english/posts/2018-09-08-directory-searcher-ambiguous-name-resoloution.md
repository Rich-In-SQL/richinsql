---
title: 'Directory Searcher & Ambiguous Name Resoloution'
date: 2018-09-08T12:12:57+01:00
author: Rich
layout: post
permalink: /directory-searcher-ambiguous-name-resoloution
image: /wp-content/uploads/2018/09/Directory-Searcher-Ambiguous-Name-Resoloution-1200x280.png
categories:
  - 'csharp'
tags:
  - Active Directory
  - Directory Searcher
  - Directory Services
---
Recently I was writing a login process that required me to query Active Directory, admittedly something I have never done before, I took to the internet to search for a solution on how this was going to work and came across the following query string to use within my existing class.

```
    search.Filter = "(&(objectCategory=person)(objectClass=user)(anr=" + Username + ")(!userAccountControl:1.2.840.113556.1.4.803:=2))";
```

I set to test this with multiple users, and it worked perfectly, the correct data was returned in 100% of the test cases that I had tried, so the code got moved into production and signed off.

Fast forward 6 months or so and a user logged a ticket saying they were unable to login to the application, there is a check you see to make sure that each user has a valid email address in Active Directory, if they don't they can't use the application, the user who had logged the ticket <span style="text-decoration: underline;"><strong>did</strong></span> have an email address so I debugged the code with the failing user's username and right enough the wrong details were returned from Active Directory.

### So what's the problem?

Here is the code I was using, The search filter is going to look for the following;

  * Objects category is people
  * Object class is user,
  * The account matches the username provided _(or so_ i _thought)_
  * The account is not disabled

As i was using **search.FindOne** It will only return the first result it finds.

```
    public static string Fname(string Username)
    {
        DirectoryEntry entry = new DirectoryEntry();
        DirectorySearcher search = new DirectorySearcher(entry);

        search.Filter = "(&(objectCategory=person)(objectClass=user)(anr=" + Username + ")(!userAccountControl:1.2.840.113556.1.4.803:=2))";

        search.PropertiesToLoad.Add("givenName");   // first name

        SearchResult result = search.FindOne();

        string Firstname = string.Empty;

        if (result.Properties["givenName"].Count &gt; 0)
        {
            Firstname = result.Properties["givenName"][0] as String;
        }
        return Firstname;
    }
```

anr or [Ambiguous Name Resolution](https://social.technet.microsoft.com/wiki/contents/articles/22653.active-directory-ambiguous-name-resolution.aspx) will take the string provided in the anr and search a pre-defined list of attributes (see below);

<table border="1" width="100%">
  <tr>
    <td>
      <strong>Windows Server</strong>
    </td>
    
    <td>
      <strong>2000</strong>
    </td>
    
    <td>
      <strong>AD LDS</strong>
    </td>
    
    <td>
      <strong>2003 (and R2)</strong>
    </td>
    
    <td>
      <strong>2008/2012 (and R2)</strong>
    </td>
  </tr>
  
  <tr>
    <td>
      <strong>Schema Version</strong>
    </td>
    
    <td>
      <strong>13</strong>
    </td>
    
    <td>
      <strong>All</strong>
    </td>
    
    <td>
      <strong>30, 31</strong>
    </td>
    
    <td>
      <strong>44, 47, 56, 69</strong>
    </td>
  </tr>
  
  <tr>
    <td>
      displayName
    </td>
    
    <td>
      X
    </td>
    
    <td>
      X
    </td>
    
    <td>
      X
    </td>
    
    <td>
      X
    </td>
  </tr>
  
  <tr>
    <td>
      givenName (First Name)
    </td>
    
    <td>
      X
    </td>
    
    <td>
    </td>
    
    <td>
      X
    </td>
    
    <td>
      X
    </td>
  </tr>
  
  <tr>
    <td>
      legacyExchangeDN
    </td>
    
    <td>
      X
    </td>
    
    <td>
    </td>
    
    <td>
      X
    </td>
    
    <td>
      X
    </td>
  </tr>
  
  <tr>
    <td>
      msDS-AdditionalSamAccountName
    </td>
    
    <td>
    </td>
    
    <td>
    </td>
    
    <td>
      X
    </td>
    
    <td>
      X
    </td>
  </tr>
  
  <tr>
    <td>
      msDS-PhoneticCompanyName
    </td>
    
    <td>
    </td>
    
    <td>
    </td>
    
    <td>
    </td>
    
    <td>
      X
    </td>
  </tr>
  
  <tr>
    <td>
      msDS-PhoneticDepartment
    </td>
    
    <td>
    </td>
    
    <td>
    </td>
    
    <td>
    </td>
    
    <td>
      X
    </td>
  </tr>
  
  <tr>
    <td>
      msDS-PhoneticDisplayName
    </td>
    
    <td>
    </td>
    
    <td>
    </td>
    
    <td>
    </td>
    
    <td>
      X
    </td>
  </tr>
  
  <tr>
    <td>
      msDS-PhoneticFirstName
    </td>
    
    <td>
    </td>
    
    <td>
    </td>
    
    <td>
    </td>
    
    <td>
      X
    </td>
  </tr>
  
  <tr>
    <td>
      msDS-PhoneticLastName
    </td>
    
    <td>
    </td>
    
    <td>
    </td>
    
    <td>
    </td>
    
    <td>
      X
    </td>
  </tr>
  
  <tr>
    <td>
      Name (RDN)
    </td>
    
    <td>
      X
    </td>
    
    <td>
      X
    </td>
    
    <td>
      X
    </td>
    
    <td>
      X
    </td>
  </tr>
  
  <tr>
    <td>
      physicalDeliveryOfficeName
    </td>
    
    <td>
      X
    </td>
    
    <td>
      X
    </td>
    
    <td>
      X
    </td>
    
    <td>
      X
    </td>
  </tr>
  
  <tr>
    <td>
      proxyAddresses
    </td>
    
    <td>
      X
    </td>
    
    <td>
      X
    </td>
    
    <td>
      X
    </td>
    
    <td>
      X
    </td>
  </tr>
  
  <tr>
    <td>
      sAMAccountName
    </td>
    
    <td>
      X
    </td>
    
    <td>
    </td>
    
    <td>
      X
    </td>
    
    <td>
      X
    </td>
  </tr>
  
  <tr>
    <td>
      sn (Last Name)
    </td>
    
    <td>
      X
    </td>
    
    <td>
    </td>
    
    <td>
      X
    </td>
    
    <td>
      X
    </td>
  </tr>
  
  <tr>
    <td>
      mail
    </td>
    
    <td>
      X
    </td>
    
    <td>
      X
    </td>
    
    <td>
      X
    </td>
    
    <td>
      X
    </td>
  </tr>
  
  <tr>
    <td>
      mailNickname
    </td>
    
    <td>
      X
    </td>
    
    <td>
      X
    </td>
    
    <td>
      X
    </td>
    
    <td>
      X
    </td>
  </tr>
  
  <tr>
    <td>
      msExchResourceSearchProperties
    </td>
    
    <td>
      X
    </td>
    
    <td>
      X
    </td>
    
    <td>
      X
    </td>
    
    <td>
      X
    </td>
  </tr>
</table>

Source: [Active Directory: Ambiguous Name Resolution](https://social.technet.microsoft.com/wiki/contents/articles/22653.active-directory-ambiguous-name-resolution.aspx)

So let's suppose the username the user had entered was bloggs the anr would look like this anr=&#8221;bloggs&#8221; the problem is it would only return the first result where &#8220;bloggs&#8221; appears at the start of any of the naming attributes listed in the pre-defined list above. This was a problem, I had made a mistake in compiling the query string.

What I actually wanted was;

  * Object Category equals person
  * Object Class equals user
  * sAMAccounName equals the provided string
  * Account is not disabled

This required a small change to the query string so instead of using anr we would need to swap this for **sAMAccountName** this will now return objects that match the above criteria but the Active Directory username must explicilty match the provided string. So if the Active Directory username was bloggsj the username in the application would have to send bloggsj else no match would be found and login would fail.

```
    public static string Fname(string Username)
    {
        DirectoryEntry entry = new DirectoryEntry();
        DirectorySearcher search = new DirectorySearcher(entry);

        search.Filter = "(&(objectCategory=person)(objectClass=user)(sAMAccountName=" + Username + ")(!userAccountControl:1.2.840.113556.1.4.803:=2))";

        search.PropertiesToLoad.Add("givenName");   // first name

        SearchResult result = search.FindOne();

        string Firstname = string.Empty;

        if (result.Properties["givenName"].Count &gt; 0)
        {
            Firstname = result.Properties["givenName"][0] as String;
        }
        return Firstname;
    }
```

### We have go to test this!

I debugged the amended code using the same username of the person who had logged the ticket, right enough the correct information was now being returned, but what about people who were not affected before the change? I tested with them too and all of the accounts returned the expected data from Active Directory.

### Trip & Fall

This tripped me up, I fell, but I got back up, debugged the problem and came up with a resoloution, I am not by any means a Senior developer so I wanted to share my findings with you guys so that if you come across this problem in your code it dosen't trip you up too, hopefully someone somewhere will find this useful.