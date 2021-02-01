# Contents

- [CORS](#CORS)

# CORS
*CORS* disallow to load resources or scripts from another domens  
if server doesn't explicitly allowed it.  
E.g. if `https://domainA.org` has a static image, it can't be  
requested from `https://domainB.org` if server with the image  
doesn't provide `Control-Allow-Origin: https://domainB.org` header.  
However, following restriction in header is a kind of agreement  
between drowser developers. Simple `curl` request will bypass *CORS*.
