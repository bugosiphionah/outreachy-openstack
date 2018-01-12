## Improve keystone Authenticaton in kubernetes

### Problem Description

Currently, the openstack cloud provider authetincates users correctly with keystone but this suthentication is not used else where when working with openstack resources instead general credentials are used .

We therefore need to ensure users are authenticated with keystone and these autheticated users used to work with openstack resources.

### Proposed changes

+ create token when authenticating with keystone. Something similar to what is achieved here :

      func (*tokenGetter) Token() (string, error) {
        options, err := openstack.AuthOptionsFromEnv()
        if err != nil {
          return "", fmt.Errorf("failed to read openstack env vars: %s", err)
        }
        client, err := openstack.AuthenticatedClient(options)
        if err != nil {
          return "", fmt.Errorf("authentication failed: %s", err)
        }
        return client.TokenID, nil
      }
  
+ Validate and use a user token to access openstack reources in kubernetes.

### Change Effects

+ Documentation

The documentation specific to the openstack cloud provider in kubernetes may need to be updated.

+ Tests

There are tests for the openstack provider in kubernetes however token specific modifications may be made.
