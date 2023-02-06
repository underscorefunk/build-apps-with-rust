# Server

## What is a server and client. Why does the divide exist?



## An analogy to understand the server and client

- Wear house of tools that we load up into our trucks before going to a job site. 
- We have two job sites. 
- There are some tools that we use at one and not the other
- There are some tools that we bring to both
- There are some tools that we call the same thing at both, but the way they function might be different. We can give instructions that talk about using the same named tool at both job sites. This is whats we're referring to when we mean #isomorphic.

>Isomorphic: corresponding or similar in form and relations. #isomorphic

### The analogy applied to web technology

Our Rust project's `src` folder is the wear house in our analogy. The `server` and `client` are two job sites where work (computation) will be done. The trucks being sent to the job sites full of purpose selected tools and instructions are `distributions`. In the case of Rust we compile a binary for distribution (sending to) the `server` and we compile `WASM` as part of the distribution for the web `client`/browser. I say part of because we include a few other static files to help setup our distribution at that job site (load our WASM in a browser).

Where things get interesting is that the `server` is like a closed job site. There's a fence around it. The `client` is like an open job site. It's less open thanks to WASM being compiled, but it's still a place where we invite the public (users) to participate. The `client` job site might need some prefabricated constructions that only the closed job site can create. It's able to make those requests, but the closed job site will take those requests and vet them first before passing them to their workers. The `client` job site will have to wait for what they need. This exemplifies the request response pattern of server/client. In the case of our application, the public is able to request 




Our server is built in Rust. It

and runs as a binary executable on a computer in a wear house or as an executable in a network of computers like Cloudflare workers.