# Adding Refreshable Globals in Postman

Setting Up Automatically Refreshable Variables from the Response in Pre-request Script

1. Create your environment in the local environment.
2. Create variables in the local environment and save them.
   <img width="740" alt="Screenshot 2024-03-15 at 17 41 58" src="https://github.com/beardmikle/postman_access_token/assets/11380960/72c88028-f384-47b9-af59-98d68bc18ef5">
4. Add the code to the Pre-request Script.
   <img width="610" alt="Screenshot 2024-03-15 at 17 46 17" src="https://github.com/beardmikle/postman_access_token/assets/11380960/c18aa935-d106-4e69-a450-d6d275abe1b4">
6. In the Authorization tab for subsequent requests, you can add a variable like {{ACCESS_TOKEN}}.
7. Save the request settings and apply.
   <img width="926" alt="Screenshot 2024-03-15 at 17 46 24" src="https://github.com/beardmikle/postman_access_token/assets/11380960/2778f27b-6091-4840-b8f4-35088251eaa6">



```javascript
const value = (key, defaultValue) => pm.globals.get(key) || defaultValue;
const setValue = (key, value) => pm.globals.set(key, value);
const getTimeStamp = (expires_in = 0) => {
    const expiryDate = new Date();
    expiryDate.setSeconds(expiryDate.getSeconds() + expires_in);
    return expiryDate.getTime();
}

const sendRequest = (urlencoded) => {
    const request = {
        url: `${pm.environment.get('base_url_8080')}/api/token`,
        method: 'POST',
        header: {
            'contetn-type': 'application/x-www-form-urlencoded'
        },
        body: { mode: 'urlencoded', urlencoded }
    };
    pm.sendRequest(request, (err, res) => {
        const { access_token, refresh_token, expires_in } = res.json();
        setValue('ACCESS_TOKEN', access_token);
        setValue('REFRESH_TOKEN', refresh_token);
        setValue('ACCESS_TOKEN_EXPIRY', getTimeStamp(expires_in));
    });
}

const sendLoginRequest = () => {
    const username = pm.environment.get('AUTH_USERNAME') || 'admin';
    const password = pm.environment.get('AUTH_PASSWORD') || 'password123';
    console.log('Send login request');
    sendRequest([
        { key: 'grant_type', value: 'password' },
        { key: 'username', value: username },
        { key: 'password', value: password }
    ]);
}

const sendRefreshTokenRequest = () => {
    const refreshToken = value('REFRESH_TOKEN');
    console.log('Send refresh token request');
    sendRequest([
        { key: 'grant_type', value: 'refresh_token' },
        { key: 'refresh_token', value: refreshToken }
    ]);
}

const token = value('ACCESS_TOKEN');
const accessTokenExpiry = value('ACCESS_TOKEN_EXPIRY');

if(!token) {
    sendLoginRequest();
} else if(accessTokenExpiry <= getTimeStamp()) {
    sendRefreshTokenRequest();
}
