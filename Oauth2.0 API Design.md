1. Define Accesset
 user auth data: uid, username, password, user pic, user phone, user email, user security questions.
 user login devices/services: service name, last login, authorized when, last ip location.

 user profile data: user credits, user groups, other not so important data.
 user pm data: user published pms.
 user notification data: user received notifications.

 user thread data: user published threads.
 user reply data: user replies.
 
 user favorite data: user faved items, forum, thread, users.
 user black list data: user blacked listed other users.
 user history data: user view thread history.
 user search data: user searched terms.
 
2. Define Oauth 2.0 Scopes
 read_profile: uid, username, user pic
 read_detail_profile: uid, username, user pic, user email, user credits, user groups
 
3. Define Oauth 2.0 authorize flow
 support Auth code flow
 support implicit flow

4. Define Oauth 2.0 Resource Access Flow
 check client, check token, check token-client relationship, check permission

5. 