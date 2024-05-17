# Phonenet integration

**1. Javascript sip call**
- Use jssip lib from: https://jssip.net/download/
- Declare connection params:

<pre><code>
let host = '[your account name].phonenet.io';
let ext = '10000';
let password = 'matkhau123'; //Your extension acc and password
let contact_uri = 'sip:'+ext+'@'+Math.random().toString(36).substring(2, 15)+'.invalid;transport=ws';
let phone = null;
let call = null;

let callOptions = {
    mediaConstraints: {
        audio: true,
        video: false
    },
    pcConfig: {
        iceServers: [
            {urls: ['stun:stun.l.google.com:19302']},
            {
                urls: 'turn:numb.viagenie.ca',
                'username': 'webrtc@live.com',
                'credential': 'muazkh'
            }
        ],
    }
};
</code></pre>

- Sip login to Phonenet:
<pre><code>


phone = new JsSIP.UA({
    sockets: [new JsSIP.WebSocketInterface('wss://' + host + ':7443')],
    uri: 'sip:' + ext + '@' + host,
    realm: host,
    ha1: md5(ext+':'+host+':'+password),
    contact_uri: contact_uri,
    session_timers: false,
});

phone.start();
</code></pre>

- Handle connect and disconnect event

<pre><code>
phone.on('registered', (e) => {
    //Connected
    console.log('Phone registered');
    console.log(e);
});

phone.on('disconnected', () => {
    //Disconnected
    console.log('Phone disconnected');
});
</code></pre>

- Handle calls event (new incoming calls and call status):

<pre><code>
phone.on("newRTCSession", (data) => {
    call = data.session;
    if (call.direction === "incoming") {
        //New incoming call
        $('#status').text('Có cuộc gọi đến, bấm trả lời để nhận');
        $('#answer').click(() => {
            call.answer(callOptions);
            call.connection.onaddstream = function (e) {
                const remoteAudio = document.createElement('audio');
                remoteAudio.srcObject = e.stream;
                remoteAudio.play();
            };
        });
    }
    call.on("progress", function () {
        console.log('progress');
    });
    call.on("accepted", function () {
        console.log('accepted');
    });
    call.on("confirmed", function () {
        console.log('confirmed');
    });
    call.on("ended", function () {
        console.log('ended');
    });
    call.on("failed", function (e) {
        console.log('failed');
        console.log(e);
    });
    console.log(this.call);
});
</code></pre>

- Make a call:

<pre><code>
let number = '0999999999';
call = phone.call(number, callOptions);
call.connection.onaddstream = function (e) {
    const remoteAudio = document.createElement('audio');
    remoteAudio.srcObject = e.stream;
    remoteAudio.play();
};
</code></pre>

**2. Client API**
- Client api url: https://clientapi.phonenet.io
- Login to client dashboard (client.phonenet.io) to get token.
- Make REST request with content type: application/json, add header token = $token
<pre><code>
let serviceUrl = 'https://clientapi.phonenet.io';
let token = '43757843yuiwqhrui3wtri534y5382yruiery34'

const service = axios.create({
    baseURL: serviceUrl,
    timeout: 30000,
});
service.defaults.headers.common['token'] = token;
</code></pre>

**User API**
- Search user: GET to /user with query strings: 

Request query string: /user?page=1&pageSize=100&keyword=

| param    | type   | required | example | desc                       |
|----------|--------|----------|---------|----------------------------|
| page     | int    | yes      | 1       | number of page             |
| pageSize | int    | yes      | 100     | number of records per page |
| keyword  | string | no       | foo     | search keyword             |

Response:

<pre><code>
{"docs":[{
        "active":true,"role":"owner",
        "createTime":1579714789883,
        "_id":"5e2888f8ef0c4c35b81e7019",
        "email":"phugt@gadget.com.vn",
        "extPassword":"matkhau123",
        "name":"Phú",
        "phone":"0977805845",
        "ext":10000
    }],"totalDocs":1,"limit":100,"totalPages":1,"page":1
}    
</code></pre>

- Create user: POST to /user with json body

| param       | type    | required | example       | desc                       |
|-------------|---------|----------|---------------|----------------------------|
| email       | string  | yes      | foo@bar.com   | email, unique              |
| name        | string  | yes      | foo           | name of user               |
| ext         | int     | yes      | foo           | sip extension number       |
| extPassword | string  | yes      | dsfhdsfe7466  | sip password               |
| password    | string  | yes      | jnfdjgfdhgryt | login password             |
| phone       | number  | yes      | 0999999999    | user phone number          |
| desc        | string  | no       | this is a men | user description           |
| role        | string  | yes      | agent         | roles: owner, admin, agent |
| group       | string  | no       |               | group id                   |
| active      | bool    | no       | true          | active user or no          |

Response:

| status        | status code | response body |
|---------------|-------------|---------------|
| success       | 200         | true          |
| invalid input | 422         | err detail    |
| internal err  | 500         | err detail    |

- Update user: PUT to /user/:id, request and response params same as create
- Delete user: DELETE to /user/:id
- Reset password: DELETE to /user/:id/password, new password will random and send to user email

**User group api**
- Search group: GET to /user-group, params same as user api
- Create group: POST to /user-group with json body

Request

| param  | type    | required | example       | desc                        |
|--------|---------|----------|---------------|-----------------------------|
| name   | string  | yes      | foo           | name of group               |
| ext    | int     | yes      | foo           | group call extension number |
| desc   | string  | no       | this is a men | group description           |
| active | bool    | no       | true          | active group or no          |

Response same as user

- Update group: PUT to /user-group/:id
- Delete group: DELETE to /user-group/:id

**Call detail record (CDR) API**
- Search CDR: GET to /cdr?page=1&pageSize=100&group=[group id]&keyword=test
- Example of a CDR params

**3. Javascript sip video call**
- Use jssip lib from: https://jssip.net/download/

- HTML:
<pre><code>
    &lt;video id=&quot;selfView&quot; autoplay muted=true&gt;&lt;/video&gt; 
    &lt;video id=&quot;remoteView&quot; autoplay&gt;&lt;/video&gt;
</code></pre>

- Declare connection params:
<pre><code>
var selfView = document.getElementById('selfView');
var remoteView = document.getElementById('remoteView');

let host = '[your account name].phonenet.io';
let ext = '10000';
let password = 'matkhau123'; //Your extension acc and password
let contact_uri = 'sip:'+ext+'@'+Math.random().toString(36).substring(2, 15)+'.invalid;transport=ws';
let phone = null;
let call = null;

let callOptions = {
    mediaConstraints: {
        audio: true,
        video: true
    },
    extraHeaders: ["X-Videocall: true"]
};
</code></pre>

- Sip login to Phonenet:
<pre><code>
phone = new JsSIP.UA({
    sockets: [new JsSIP.WebSocketInterface('wss://' + host + ':7443')],
    uri: 'sip:' + ext + '@' + host,
    realm: host,
    ha1: md5(ext+':'+host+':'+password),
    contact_uri: contact_uri,
    session_timers: false,
});

phone.start();
</code></pre>

- Handle connect and disconnect event

<pre><code>
phone.on('registered', (e) => {
    //Connected
    console.log('Phone registered');
    console.log(e);
});

phone.on('disconnected', () => {
    //Disconnected
    console.log('Phone disconnected');
});
</code></pre>

- Handle calls event (new incoming calls and call status):

<pre><code>
phone.on("newRTCSession", (data) => {
    call = data.session;
    if (call.direction === "incoming") {
        //check video call
        let isVideo = data.request.headers['X-Videocall'] && data.request.headers['X-Videocall'].length;
        //New incoming call
        $('#status').text('Có cuộc gọi đến, bấm trả lời để nhận');
        $('#answer').click(() => {
            call.answer(callOptions);
            call.connection.onaddstream = function (e) {
                const remoteAudio = document.createElement('audio');
                remoteAudio.srcObject = e.stream;
                remoteAudio.play();
                
                //video
                remoteView.srcObject = (e.stream);
                selfView.srcObject = (session.connection.getLocalStreams()[0]);
            };
        });
    }
    call.on("progress", function () {
        console.log('progress');
    });
    call.on("accepted", function () {
        console.log('accepted');
    });
    call.on("confirmed", function () {
        console.log('confirmed');
    });
    call.on("ended", function () {
        console.log('ended');
    });
    call.on("failed", function (e) {
        console.log('failed');
        console.log(e);
    });
    console.log(this.call);
});
</code></pre>

- Make a call:

<pre><code>
let number = '20000'; //Supports internal video calls (To employee ext)
call = phone.call(number, callOptions);
call.connection.onaddstream = function (e) {
    const remoteAudio = document.createElement('audio');
    remoteAudio.srcObject = e.stream;
    remoteAudio.play();
    
    //video
    remoteView.srcObject = (e.stream);
    selfView.srcObject = (session.connection.getLocalStreams()[0]);
};
</code></pre>