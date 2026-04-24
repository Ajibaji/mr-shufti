# Mr-Shufti

_Shufti_ - a British informal, slang term for a quick look, glance, or inspection, often used in the phrase "have a shufti". Originating from Arabic (meaning "have you seen?") and adopted by British soldiers in the Middle East during World War II, it implies taking a brief look or surveying a scene

Another little tool to aid local development of Kubernetes based services. Develop a single service locally against the balance of services running in the kubernetes cluster. No more docker-compose for local development. Clusters setup with ephemeral environments/namespaces are ideal for this development approach.

## Features
For a chosen namespace:
 - port-forward all services to a local port
 - stream logs for remote services to local console
 - open chrome with a custom hosts map
     - services can be addressed using their service name
     - frontends running in the namespace can be accessed

Mr-Shufti is auto-updating

## Install
```curl https://raw.githubusercontent.com/Ajibaji/mr-shufti/master/install | bash```
