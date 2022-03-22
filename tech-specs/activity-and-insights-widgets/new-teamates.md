# Tech Spec: New Teamates Widget

![](./screenshots/new-team-members.png)

## Outline
The component shows a count of new teammates in the last X number of days, 5 of the newest members and a button to expand the view and see the rest.

Need a design to see what the “see all” looks like.

## Performance and Scalability
I don’t think this will be particularly troublesome for performance.

## Websockets

## Rest APIs
We need to add a new field to teammembers in order to record the date people joined a team.

```
GET /api/v4/teammembers?page=0&per_page=60&member_since=<timestamp>&team_id=123
```
```json
[
    {
        "team_id": "123",
        "user_id": "string",
        ...otherTeamMemberInfo,
        "member_since": 1646571442991,
    },
    ...
]
```

## Plugins

## Algorithm

## Temporal
- Last 24 hours
- Last 7 days
- Last 28 days

## Scope
- Scope by team

## Authorization
Does each individual widget require a permission?

## Guest Visibility
This is up for debate, guests are only granted access to certain members so they should only be able to see 

## Licensing
I’m guessing the overarching feature of insights will have a license and each widget will be the same license type?

## Configs/Feature Flags
This widget will need it’s own config to enable/disable it

## Database Schema Changes
Need to add a new field to teammembers to record the date a user joined a given team
