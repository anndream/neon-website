---
title: Organizations and Teams
subtitle: Managing teams and team members in Neon Auth
enableTableOfContents: true
tag: beta
---

Teams provide a structured way to group users and manage their permissions. Users can belong to multiple teams simultaneously, allowing them to represent departments, B2B customers, or projects.

The server can perform all operations on a team, but the client can only carry out some actions if the user has the necessary permissions. This applies to all actions that can be performed on a server/client-side `User` object and a `Team` object.

## Concepts

### Team permissions

If you attempt to perform an action without the necessary team permissions, the function will throw an error. Always check if the user has the required permission before performing any action. Learn more about permissions [here](/concepts/permissions).

Here is an example of how to check if a user has a specific permission on the client:

```tsx shouldWrap
const user = useUser({ or: 'redirect' });
const team = user.useTeam('some-team-id');

if (!team) {
  return <div>Team not found</div>;
}

const hasPermission = user.usePermission(team, '$invite_members');

if (!hasPermission) {
  return <div>No permission</div>;
}

// Perform corresponding action like inviting a user
```

### Team profile

A user can have a different profile for each team they belong to (Note this is different to the user's personal profile). This profile contains information like `displayName` and `profileImageUrl`. The team profile can be left empty and it will automatically take the user's personal profile information.

The team profile is visible to all the other users in the team that have the `$read_members` permission.

## Retrieving a user's teams

You can list all teams a user belongs to using the `listTeams` or `useTeams` functions or fetch a specific team with `getTeam` or `useTeam`. These functions work on both clients and servers.

<Tabs labels={["Client Component", "Server Component"]}>

<TabItem>
```tsx shouldWrap
const user = useUser({ or: 'redirect' });
const allTeams = user.useTeams();
const someTeam = user.useTeam('some-team-id'); // May be null if the user is not a member of this team

return (

  <div>
    {allTeams.map(team => (
      <div key={team.id}>{team.displayName}</div>
    ))}
  </div>
  <div>
    {someTeam ? someTeam.displayName : 'Not a member of this team'}
  </div>
);
```
</TabItem>

<TabItem>
```tsx shouldWrap
const user = await stackServerApp.getUser({ or: 'redirect' });
const allTeams = await user.listTeams();
const someTeam = await user.getTeam('some-team-id'); // May be null if the user is not a member of this team

return (

  <div>
    {allTeams.map(team => (
      <div key={team.id}>{team.displayName}</div>
    ))}
  </div>
  <div>
    {someTeam ? someTeam.displayName : 'Not a member of this team'}
  </div>
);
```
</TabItem>

</Tabs>

## Creating a team

To create a team, use the `createTeam` function on the `User` object. The user will be added to the team with the default team creator permissions.

```tsx shouldWrap
const team = await user.createTeam({
  displayName: 'New Team',
});
```

To create a team on the server without adding a specific user, use the `createTeam` function on the `ServerApp` object:

```tsx shouldWrap
const team = await stackServerApp.createTeam({
  displayName: 'New Team',
});
```

## Updating a team

You can update a team with the `update` function on the `Team` object.

On the client, the user must have the `$update_team` permission to perform this action.

```tsx shouldWrap
await team.update({
  displayName: 'New Name',
});
```

## Custom team metadata

You can store custom metadata on a team object, similar to the user object. The metadata can be any JSON object.

- `clientMetadata`: Can be read and updated on both the client and server sides.
- `serverMetadata`: Can only be read and updated on the server side.
- `clientReadOnlyMetadata`: Can be read on both the client and server sides, but can only be updated on the server side.

```tsx shouldWrap
await team.update({
  clientMetadata: {
    customField: 'value',
  },
});

console.log(team.clientMetadata.customField); // 'value'
```

## List users in a team

You can list all users in a team with the `listUsers` function or the `useUsers` hook on the `Team` object. Note that if you want to get the team profile, you need to get it with `user.teamProfile`.

On the client, the current user must have the `$read_members` permission in the team to perform this action.

<Tabs labels={["Client Component", "Server Component"]}>

<TabItem>
```tsx shouldWrap
// ... retrieve the team and ensure user has the necessary permissions

const users = team.useUsers();

return (

  <div>
    {users.map(user => (
      <div key={user.id}>{user.teamProfile.displayName}</div>
    ))}
  </div>
);
```
</TabItem>

<TabItem>
```tsx shouldWrap
// ... retrieve the team

const users = await team.listUsers();

return (

  <div>
    {users.map(user => (
      <div key={user.id}>{user.teamProfile.displayName}</div>
    ))}
  </div>
);
```
</TabItem>

</Tabs>

## Get current user's team profile

You can get the current user's team profile with the `getTeamProfile` or `useTeamProfile` function on the `User` object. This function returns the team profile for the team with the given ID.

<Tabs labels={["Client Component", "Server Component"]}>

<TabItem>
```tsx shouldWrap
const teamProfile = user.useTeamProfile(team);
```
</TabItem>

<TabItem>
```tsx shouldWrap
const teamProfile = await user.getTeamProfile(team);
```
</TabItem>

</Tabs>

## Invite a user to a team

You can invite a user to a team using the `inviteUser` function on the `Team` object. The user will receive an email with a link to join the team.

On the client side, the current user must have the `$invite_members` permission to perform this action.

```tsx shouldWrap
await team.inviteUser(email);
```

## Adding a user to a team

If you want to add a user to a team without sending an email, use the `addUser` function on the `ServerTeam` object. This function can only be called on the server side.

```tsx shouldWrap
await team.addUser(user.id);
```

## Removing a user from a team

You can remove a user from a team with the `removeUser` function on the `Team` object.

On the client side, the current user must have the `$remove_members` permission to perform this action.

```tsx shouldWrap
await team.removeUser(user.id);
```

## Leaving a team

All users can leave a team without any permissions required.

```tsx shouldWrap
const team = await user.getTeam('some-team-id');
await user.leaveTeam(team);
```

## Deleting a team

You can delete a team with the `delete` function on the `Team` object.

On the client side, the current user must have the `$delete_team` permission to perform this action.

```tsx shouldWrap
await team.delete();
```

<Admonition type="note">
  Team creation and management is handled through the Neon Auth API. All team operations are performed programmatically using the provided functions and hooks.
</Admonition>
