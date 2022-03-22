# Tech Spec: Top Reactions Widget

![](./screenshots/top-reactions.png)

## Outline
This component shows the top reactions for a given team or user.

## Performance and Scalability
This will have the same performance concerns as top channels and top posts/threads where it may be beneficial to cache the data and only fetch new data every X minutes.

There are concerns regarding query performance when we do fetch the data for a team. Reactions are team agnostic, so we will need to reference a reaction's parent post and channel to then filter by team.
- Reactions will need an index on `createat`.

## Websockets

## Rest APIs
New endpoint to get top reactions (for a given team and user).


### Team
```
GET /api/v4/reactions/top?page=0&per_page=5&since=<timestamp>&team_id=team123
```
```
[
	{
        emoji_name: 'smile',
        emoji_count: 100,
        ...otherReactionInfo,
	},
    ...
]
```

### User
```
GET /api/v4/reactions/top?page=0&per_page=5&since=<timestamp>&user_id=user123
```
```
[
	{
        emoji_name: 'smile',
        emoji_count: 100,
        ...otherReactionInfo,
	},
    ...
]
```

## Plugins


## Algorithm

### Queries
```sql
-- Top 5 Reactions for a Team
SELECT
	emojiname,
	count(emojiname) AS emoji_count
FROM
	reactions r
	INNER JOIN posts p ON r.postid = p.id
	INNER JOIN channels c ON p.channelid = c.id
	INNER JOIN teams t ON c.teamid = t.id
WHERE
	r.deleteat = 0
	AND t.id = 'team123'
	AND r.createat > 1646923727584
GROUP BY
	r.emojiname
ORDER BY
	emoji_count DESC
LIMIT 5
```

```sql
-- Top 5 Reactions for a User
SELECT
	emojiname,
	count(emojiname) AS emoji_count
FROM
	reactions r
WHERE
	r.deleteat = 0
	AND r.userid = 'user123'
	AND r.createat > 1646923727584
GROUP BY
	r.emojiname
ORDER BY
	emoji_count DESC
LIMIT 5
```

## Design Patterns That Could be Used

## Database Schema Changes


## Temporal
- Last 24 hours
- Last 7 days
- Last 28 days

## Scope
- Top reactions across the team (including public/private channels)
- Top reactions for the current user across the workspace (including public/private/DM/GM channels)

## Authorization
- Does each individual widget require a permission?
- Should have same authorization requirements as top posts/threads

## Guest Visibility
This is up for debate, guests are only granted access to certain channels so should they be able to see reaction info for other channels? The reaction information that is exposed via this widget is minimal so it's up for discussion if it's worth filtering the data further for guest accounts.

## Licensing
- Match overall feature license

## Configs/Feature Flags
This widget will need it’s own config to enable/disable it.

## Common Components
- The widget (and its inner elements) will be common components shared across the insights component.
- The bar graph used for the top 5 reactions is shared with the top channels bar graph.

## Security Review

## Telemetry
