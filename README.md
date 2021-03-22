# Keycloak NodeJS Adapter demo

## Setup

```sh
git clone https://github.com/nmasse-itix/keycloak-nodejs-demo.git
cd keycloak-nodejs-demo
npm install
```

## Demo scenario

Install Red Hat SSO.

Create a Realm named Red Hat.

Start the Petstore Server.

```sh
node index.js
```

Show some REST requests.

```sh
http http://localhost:8080/pets/ 
http http://localhost:8080/pets/1
```

Create a client named "nodejs-client", **Bearer Only**.

Download **keycloak.json** from the **Installation** tab (select **Keycloak OIDC JSON** from the dropdown list).

Uncomment all lines in index.js and restart the Petstore server.

Show that the Petstore server now requires authentication.

```sh
http http://localhost:8080/pets/ 
```

Create a **confidential** client named "rest-client", with only the **Direct Access Grants** flow enabled.

Create a user **john** with password **secret**.

Request a token for john.

```sh
curl https://$SSO_HOSTNAME/auth/realms/redhat/protocol/openid-connect/token -XPOST -d client_id=rest-client -d client_secret=$CLIENT_SECRET -d grant_type=password -d username=john -d password=secret 
```

Save it for later.

```sh
TOKEN=$(curl https://$SSO_HOSTNAME/auth/realms/redhat/protocol/openid-connect/token -XPOST -d client_id=rest-client -d client_secret=$CLIENT_SECRET -d grant_type=password -d username=john -d password=secret -s |jq -r .access_token)
```

Call one of the REST endpoints.

```sh
http http://localhost:8080/pets/ "Authorization:Bearer $TOKEN"
```

Now edit index.js and update all REST endpoint to check for user roles, either **read** or **write**. Like this:

```js
router.get("/pets/:id", keycloak.protect("read"), function(req,res){
```

Restart the petstore server and get a new token.

Show that now, calls are rejected.

```sh
http http://localhost:8080/pets/ "Authorization:Bearer $TOKEN"
```

Give the **read** role to john, get a new token and show that you can query the Read REST endpoints.

```sh
http http://localhost:8080/pets/ "Authorization:Bearer $TOKEN"
```

Write calls a forbidden.

```sh
http DELETE http://localhost:8080/pets/1 "Authorization:Bearer $TOKEN"
```

Give the **write** role to john, get a new token and show that you can query the Write REST endpoints.

