# Real Deal HTML

Simple solve, we can look at the source of the html to find the flag commented

bcactf{tH4TZ_D4_R34l_D3Al_cb8949}

# Agent Rocket

Logging screen with details outlined in the source commented out.
Followed by an empty screen prompting to use a "control panel" to access the flag.
The name of the device is outlined in the comments, hinting at changing the user-agent
Used burp to change the request

bcactf{u53r_4g3Nt5_5rE_c0OL_1023}

# Three Step Trivia

1st step involves using burp to bypass the frontend restriction of no decimals in the input
2nd step involves using the url to input our answer
3rd step involves chaning the DOM and having the button turn from hidden to visible.

# Cookies

Website has an admin panel
Access admin panel where the username is readonly, but the password is editable.
Looking at the adminLog.js, a correct login with admin redirects to /adminEditor.html.
No authorisation, so accessing that subdomain is possible but presents an empty page.
The source code has a file called editor.js

```
if (getCookie("pwd") == "98e99e97e99e116e102e123e117e36e101e114e115e95e115e51e51e95e99e48e48e107e33e101e115e95e55e111e111e95e56e54e51e111e52e116e53e125e") {
    window.location.replace("flag.html");
}
```

And so changing the cookie with key 'pwd' and the needed value redirects to flag.html.

bcactf{u$ers_s33_c00k!es_7oo_863o4t5}

# BCACTF 3.0: Story Mode

A webpage that dynamically loaded its contents through clj files.
Looking at core.clj which was provided,

```clj
(ns story-mode.core
  (:gen-class)
  (:require
   [aleph.http :as http]
   [clojure.java.io :as io]
   [compojure.core :as compojure]
   [compojure.route :as route]
   [ring.middleware.params :as params]
   [ring.util.request :as request]
   [ring.util.response :as response]))

(defn check-flag [req]
  (if (= (request/body-string req) (slurp "flag.txt"))
    {:status 200 :body "yes"}
    {:status 400 :body "no"}))

(defn get-story-part [req]
  (let [part ((req :query-params) "part")]
    (if (nil? part)
      {:status 400 :body "No part specified"}
      (response/file-response (str "story/" part)))))

(def handle
  (compojure/routes
    (compojure/GET "/" []
                   {
                    :status 200
                    :headers {"content-type" "text/html"}
                    :body (io/input-stream (io/resource "public/index.html"))
                   })
    (compojure/POST "/check" [] check-flag)
    (compojure/GET "/story" [] (params/wrap-params get-story-part))
    (route/resources "/")
    (route/not-found "hm")))

(defn -main []
  (http/start-server handle {:port 3000}))
```

check-flag initially looks like it has potential, but it simply checks if the given string in the body is equal to the flag.
The only other thing is get-story-part which is used to navigate the pages in the story.
```(response/file-response (str "story/" part)))))```
response/file-response from reading the documentation reads a file given to it.
Now we know that story/ {part} are files in the directory
And so we can inject file traversal through the part variable in the URL to access flag.txt. 
http://web.bcactf.com:49218/story?part=../flag.txt

bcactf{the_envd_jZNCCrTgxR237loZY12JlA}

# Jason's Web Tarot

Getting a card gives a token cookie. It's a jwt cookie with no signature.
First part is {"alg":"none","typ":"JWT"}
Second part is {"isSubscriber":false,"iat":1654300574}
Change isSubscriber to true
Encode to base64 
Edit your cookie making sure not to add the additional '=' character
Then get a new card to retrieve the flag

bcactf{n0_s3cr3t5????!!!?!_38893}