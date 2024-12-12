# Managing Application Configuration

Application Configuration is the process of providing dynamic values to your applications at runtime to control their behavior.
## Storing Config data

There are 2 primary way to store configuration data in k8s 
- Config Maps
- Secrets
Both can be mounted as environment variables or files within a pod
### Config Map
- Store key-value pairs of configuration data
- Ideal for non-sensitive configuration
- A sample Config Map 
	![[Pasted image 20241203222138.png]]
### Secrets
- Ensure that sensitive information is encrypted and securely stored
- Store sensitive information like passwords, API keys, and certificates.
- We have to store encoded values to base64 to the keys in secret
- A sample Secret
	![[Pasted image 20241203223715.png]]

## Passing Config data

Now that we have stored our config data how can we pass them to our application
To pass this data we have mainly 2 option
### Environment Variables
- We can set up environment variables directly within the pod specification
- Can be used for both sensitive and non-sensitive configuration
- We can define a env variable that we are pulling from a config map but it will passed as a env variable to our container process
	![[Pasted image 20241203222950.png]]
### Configurations Volumes
- Configuration Volumes are a type of volume in Kubernetes that allows you to mount configuration files into your pods. 
- In this Each top level key will appear as a file containing all keys below it.
- This is a convenient way to manage and update configuration data without rebuilding your container images.
- A Config Volume with a config map and a secret mounted to it
	![[Pasted image 20241203223509.png]]


# Managing Container Resources
## Resource Request

# Monitoring Container Health with Probes


# Building a Self Healing Pod with Restart Policies


# Creating a Multi-Container Pod


# Introduction to Init Container