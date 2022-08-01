<script setup>
  // Inspired by https://developers.google.com/identity/oauth2/web/guides/migration-to-gis#gapi-callback
  let tokenClient;
  let gapiInited;
  let gisInited;

  function checkBeforeStart() {
      if (gapiInited && gisInited){
        // Start only when both gapi and gis are initialized.
        document.getElementById("showEventsBtn").style.visibility="visible";
        document.getElementById("revokeBtn").style.visibility="visible";
      }
  }

  function gapiInit() {
    gapi.client.init({
      // NOTE: OAuth2 'scope' and 'client_id' parameters have moved to initTokenClient().
    })
    .then(function() {  // Load the Calendar API discovery document.
      gapi.client.load('https://www.googleapis.com/discovery/v1/apis/calendar/v3/rest');
      gapiInited = true;
      checkBeforeStart();
    });
  }

  function gapiLoad() {
      gapi.load('client', gapiInit)
  }

  function gisInit() {
    tokenClient = google.accounts.oauth2.initTokenClient({
              client_id: '27705022075-hpuh04is72fpl78s4au1igbm10ab4ce1.apps.googleusercontent.com',
              scope: 'https://www.googleapis.com/auth/calendar.readonly',
              callback: '',  // defined at request time
          });
    gisInited = true;
    checkBeforeStart();
  }

  function showEvents() {
    console.log("hello")

    tokenClient.callback = (resp) => {
      if (resp.error !== undefined) {
        throw(resp);
      }
      // GIS has automatically updated gapi.client with the newly issued access token.
      console.log('gapi.client access token: ' + JSON.stringify(gapi.client.getToken()));

      gapi.client.calendar.events.list({ 'calendarId': 'primary' })
      .then(calendarAPIResponse => console.log(JSON.stringify(calendarAPIResponse)))
      .catch(err => console.log(err));

      document.getElementById("showEventsBtn").innerText = "Refresh Calendar";
    }

    // Conditionally ask users to select the Google Account they'd like to use,
    // and explicitly obtain their consent to fetch their Calendar.
    // NOTE: To request an access token a user gesture is necessary.
    if (gapi.client.getToken() === null) {
      // Prompt the user to select an Google Account and asked for consent to share their data
      // when establishing a new session.
      tokenClient.requestAccessToken({prompt: 'consent'});
    } else {
      // Skip display of account chooser and consent dialog for an existing session.
      tokenClient.requestAccessToken({prompt: ''});
    }
  }

  function revokeToken() {
    let cred = gapi.client.getToken();
    if (cred !== null) {
      google.accounts.oauth2.revoke(cred.access_token, () => {console.log('Revoked: ' + cred.access_token)});
      gapi.client.setToken('');
      document.getElementById("showEventsBtn").innerText = "Show Calendar";
    }
  }

  window.addEventListener('onload', () => {
    document.getElementById("showEventsBtn").style.visibility="hidden";
    document.getElementById("revokeBtn").style.visibility="hidden";
    gapiLoad();
    gisInit();
  });
</script>

<template>
  <div class="setup">
    <h1>This is the setup page</h1>
  </div>

  <!-- <div id="g_id_onload"
    data-client_id="27705022075-hpuh04is72fpl78s4au1igbm10ab4ce1.apps.googleusercontent.com"
    data-context="signin"
    data-ux_mode="popup"
    data-callback="handleGoogleSecret"
    data-auto_prompt="false">
  </div>

  <div class="g_id_signin"
    data-type="standard"
    data-shape="rectangular"
    data-theme="outline"
    data-text="signin_with"
    data-size="large"
    data-logo_alignment="left">
  </div> -->
  <div id="buttonDiv"></div> 

  <button id="showEventsBtn" @click="showEvents">Show Calendar</button><br><br>
  <button id="revokeBtn" @click="revokeToken">Revoke access token</button>


</template>

<style>
@media (min-width: 1024px) {
  .about {
    min-height: 100vh;
    display: flex;
    align-items: center;
  }
}
</style>
