import React, { useEffect } from 'react';
import { PublicClientApplication } from "@azure/msal-browser";
import { MsalProvider, useMsal } from "@azure/msal-react";

const config = {
    auth: {
        clientId: "your_azure_app_id",
        authority: "https://login.microsoftonline.com/your_tenant_id",
        redirectUri: "your_redirect_uri",
    },
    cache: {
        cacheLocation: "localStorage",
        storeAuthStateInCookie: true
    }
};

const pca = new PublicClientApplication(config);

const handleResponse = (response) => {
    if (response !== null) {
        const account = pca.getAccountByUsername(response.account.username);
        if (account) {
            pca.setActiveAccount(account);
        }
    }
};

const request = {
    scopes: ["User.Read"]
};

// Azure AD Login
const Login = () => {
    const { instance } = useMsal();

    const handleLogin = () => {
        instance.loginPopup(request).then(handleResponse);
    };

    return (
        <button onClick={handleLogin}>Sign In</button>
    );
};

// Auto Logout after 10 mins of inactivity
const AutoLogout = () => {
    let timer;

    const resetTimer = () => {
        if (timer) {
            clearTimeout(timer);
        }
        timer = setTimeout(logout, 600000);  // 10 mins = 600000 ms
    };

    const logout = () => {
        pca.logout();
    };

    useEffect(() => {
        resetTimer();
        window.onmousemove = resetTimer;
        window.onkeypress = resetTimer;
    }, []);

    return null;
};

// Main App
const App = () => {
    useEffect(() => {
        const intervalId = setInterval(() => {
            // Refresh token every 15 mins
            pca.acquireTokenSilent(request).then(handleResponse);
        }, 900000);  // 15 mins = 900000 ms

        return () => clearInterval(intervalId);
    }, []);

    return (
        <MsalProvider instance={pca}>
            <Login />
            <AutoLogout />
        </MsalProvider>
    );
};

export default App;