# Authz
SaaS authorization model using openFGA, inspired by Google Cloud IAM.

The authorization [model](model.fga) is represented in openFGA configuration language.

## Getting Started

Below is a simple walkthrough for some of the key features of the authorization model.

To play with the examples:
1. Go to https://play.fga.dev/stores/create/?id=01K8HVGGK5S5KGR47RFNKD90RY
2. Add a store name, DEMO for example.

### Group Membership
If a user is a member of a group, and that group has access to a resource, then the user has that same access to the resource.

For example, given:

| User                 | Relation | Object          |
| -------------------- | -------- | --------------- |
| user:bob             | member   | group:app_one   |
| group:app_one#member | admin    | application:one |

Then bob is an admin of application:one, and this openFGA query returns true:
```
is user:bob an admin of application:one
```

### Admin Role
The admin role will infer all other roles.

For example, because bob is an admin, bob will also be a policy_writer and policy_reader:
```
is user:bob a policy_writer of application:one
is user:bob a policy_reader of application:one
```

### Change IAM Policies
The policy_writer role grants a user the ability to make access control changes (add/remove openFGA tuples)

### Tags
If an application and a device share the same tag, then the application can run on the device.

For example, given:

| User            | Relation    | Object          |
| --------------- | ----------- | --------------- |
| application:one | application | tag:app_one_tag |
| tag:app_one_tag | tag         | device:d1       |


Then device d1 has an application of application:one
```
is application:one an application of device:d1
```

## Concepts

### Prinicipals
A security principal is an entity that can be authenticated and granted access to resources.

Examples:
- A user account
- A computer or device
- A service or application account
- A group of users, 

### Roles

#### Basic Roles
Basic roles are highly permissive roles that give broad access to resources.

The basic roles in IAM are `admin`, `writer` and `reader`.

#### Policy Roles
The `policy_writer` role allows access control policies (IAM policies) to be placed on resources.