<pre>{"auth0: "+JSON.stringify(auth0, null, 2)}</pre>
<button id="btn-login" disabled={isAuthenticated} on:click={login}>Log in</button>
<button id="btn-logout" disabled={!isAuthenticated} on:click={logout}>Log out</button>
<pre>{JSON.stringify(user, null, 2)}</pre>
<script>
	import {onMount} from "svelte"; // onMount() is executed at the end of loading the page
	import createAuth0Client from "@auth0/auth0-spa-js"; // the auth0 library
	let isAuthenticated, user, auth0; // global vars
	const query = window.location.search; // the url content, used by auth0 after logging
	const config = { // can't paste these ones into the auth0, I don't know why
		domain: "<yourdomain>.auth0.com",
		clientId: "<yourclientid>"
	};
	onMount(async () => {
		auth0 = await createAuth0Client({ // your auth domain and client ID
			domain: config.domain,
			client_id: config.clientId
		});
		isAuthenticated = await auth0.isAuthenticated(); // check if client is already logged-in
		user = await auth0.getUser(); // check for user data
		// Only when coming back from Auth0 login, Process the "code" and "state" data from URL
		if (query.includes("code=") && query.includes("state=")) {
			await auth0.handleRedirectCallback(); // deals with code and state
			window.history.replaceState({}, document.title, "/"); // cleans the URL
			user = await auth0.getUser(); // refresh user data
			isAuthenticated = true; // sets our local indicator
		}
	});
	const login = async () => {
		if (auth0) { // to prevent SSR execution 
			try {
				await auth0.loginWithRedirect({
					redirect_uri: "http://localhost:5000" //window.location.origin
				});
			} catch (e) { // just in case
				console.error(e);
			}
		}
	};
	const logout = async () => {
		if (auth0) { // to prevent SSR execution 
			auth0.logout();
			isAuthenticated = false; // sets our local indicator
		}
	}; // based on https://auth0.com/docs/quickstart/spa/vanillajs#log-the-user-out & https://github.com/auth0-blog/svelte-auth0
</script>
