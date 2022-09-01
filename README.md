Supported tags and respective `Dockerfile` links
================================================

This is a fork from:
  * [`latest` (Dockerfile)](https://github.com/wernight/docker-dante/blob/master/Dockerfile) [![](https://images.microbadger.com/badges/image/wernight/dante.svg)](https://microbadger.com/images/wernight/dante "Get your own image badge on microbadger.com")

To build the image:

    $ cp sockd.conf.withAuthent sockd.con # to have a dante configured to only accept authenticated user
    $ docker build .
     ---> 7db37e99c701
    Step 8/8 : CMD ["sockd"]
    ---> Using cache
    ---> 9ede6c1a16fe
    Successfully built 9ede6c1a16fe

Then execute the container:

    $ docker run -d -p 1080:1080  9ede6c1a16fe
    b7f26777bb650bd28ac09ebbbff55363d0e3c4c2a22f14a3930c436c996e7870

Then test a curl call

Without authentication:

    $ curl --proxy socks5://localhost:1080  https://gist.githubusercontent.com/ypiel-talend/c3277c902b92a8e266d1eb3c0b0e576a/raw/956d70aa753af6cfa2a3a710a9150deff70b47f6/geologists_ok.json
       {
      "content":[
         {
            "name":"MauriceKrafft",
            "age":35,
            "female":false,
            "address":{
            "city":"Mulhouse",
            "zipcode":68100
         },
         ...
      ]
    }

With authentication:
- The login is "peter"
- The password is "aze123_=KLM"


    $ curl --proxy socks5://localhost:1080  https://gist.githubusercontent.com/ypiel-talend/c3277c902b92a8e266d1eb3c0b0e576a/raw/956d70aa753af6cfa2a3a710a9150deff70b47f6/geologists_ok.json
    curl: (7) No authentication method was acceptable. (It is quite likely that the SOCKS5 server wanted a username/password, since none was supplied to the server on this connection.)
    
    $ curl --proxy socks5://peter:aze123_=KLM@localhost:1080  https://gist.githubusercontent.com/ypiel-talend/c3277c902b92a8e266d1eb3c0b0e576a/raw/956d70aa753af6cfa2a3a710a9150deff70b47f6/geologists_ok.json
       {
      "content":[
         {
            "name":"MauriceKrafft",
            "age":35,
            "female":false,
            "address":{
            "city":"Mulhouse",
            "zipcode":68100
         },
         ...
      ]
    }


What is Dante
-------------

[**Dante**](http://www.inet.no/dante/index.html) consists of a SOCKS server and a SOCKS client, implementing RFC 1928 and related standards. It can in most cases be made transparent to clients, providing functionality somewhat similar to what could be described as a non-transparent Layer 4 router. For customers interested in controlling and monitoring access in or out of their network, the Dante SOCKS server can provide several benefits, including security and TCP/IP termination (no direct contact between hosts inside and outside of the customer network), resource control (bandwidth, sessions), logging (host information, data transferred), and authentication.


Usage example
-------------

    $ docker run -d -p 1080:1080 wernight/dante

Change its configuration by mounting a custom `/etc/sockd.conf`
(see [sample config files](http://www.inet.no/dante/doc/latest/config/server.html)).


### Client-side set up

Set your browser or application to use SOCKS v4 or v5 proxy `localhost` on port 1080,
like for example:

    $ curl --proxy socks5://localhost:1080 https://example.com

... or set to use PAC script like:

    function FindProxyForURL(url, host) {
      return "SOCKS localhost:1080";
    }


### Requiring authentication

The default config in this image allows everyone to use the proxy. You can add a simple authentication (which will send data unencrypted) by setting up a `Dockerfile` like:

    FROM wernight/dante

    # TODO: Replace 'john' and 'MyPassword' by any username/password you want.
    RUN printf 'MyPassword\nMyPassword\n' | adduser john

Uncomment line in `sockd.conf`:

    socksmethod: username

Then use SOCKS v5, for example:

    $ curl --proxy socks5://john:MyPassword@localhost:1080 https://example.com

Note: SOCKS v4 will be blocked.

WARNING: Many browsers do **not** support SOCKS authentication (e.g. see this [Chrome bug](https://bugs.chromium.org/p/chromium/issues/detail?id=256785)).


Feedbacks
---------

Suggestions are welcome on our [GitHub issue tracker](https://github.com/wernight/docker-dante/issues).
