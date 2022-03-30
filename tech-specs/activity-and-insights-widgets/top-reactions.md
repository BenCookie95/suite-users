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

#### Parameters:
- `time_range`:
    - `1_day`
    - `7_day`
    - `30_day`
- `page`
- `per_page`

### Team
```
GET /api/v4/reactions/top/team/<team_id>?time_range=1_day&page=0&per_page=5
```
```json
[
	{
        "emoji_name": "smile",
        "count": 100,
	},
]
```

### User

**Across the workspace**
```
GET /api/v4/reactions/top/user/<user_id>?time_range=1_day&page=0&per_page=5
```

**Scoped to team**
```
GET /api/v4/reactions/top/user/<user_id>?time_range=1_day&page=0&per_page=5&team_id=<team_id>
```

```json
[
	{
        "emoji_name": "smile",
        "count": 100,
	},
]
```

- `emoji_name` is a `string` that contains the name of the emoji.
- `count` is an `int64` and represents the number of times that emoji was used.

## Plugins


## Algorithm

### Queries
```sql
-- Top 5 Reactions for a Team
SELECT DISTINCT
	EmojiName,
	SUM(EmojiCount) OVER (PARTITION BY EmojiName) AS Count
FROM ((
		SELECT
			EmojiName,
			count(EmojiName) AS EmojiCount
		FROM
			ChannelMembers
			INNER JOIN Channels ON ChannelMembers.ChannelId = Channels.Id
			INNER JOIN Posts ON Channels.Id = Posts.ChannelId
			INNER JOIN Reactions ON Posts.Id = Reactions.PostId			
		WHERE
			ChannelMembers.UserId = ?
			AND Channels.Type = 'P'
			AND Reactions.DeleteAt = 0
			AND Channels.TeamId = ?
			AND Reactions.CreateAt > ?
		GROUP BY
			Reactions.EmojiName)
	UNION ALL (
		SELECT
			EmojiName,
			count(EmojiName) AS EmojiCount
		FROM
			Reactions
			INNER JOIN Posts ON Reactions.PostId = Posts.Id
			INNER JOIN Channels ON Posts.ChannelId = Channels.Id
		WHERE
			Reactions.DeleteAt = 0
			AND Channels.Type = 'O'
			AND Channels.TeamId = ?
			AND Reactions.CreateAt > ?
		GROUP BY
			Reactions.EmojiName)) AS A
ORDER BY
	Count DESC
LIMIT ?
OFFSET ?
```

```sql
-- Top 5 Reactions for a User (Across entire workspace incl. DMs/GMs)
SELECT
	EmojiName,
	count(EmojiName) as Count
FROM
	Reactions
WHERE
	Reactions.DeleteAt = 0
	AND Reactions.UserId = ?
	AND Reactions.CreateAt > ?
GROUP BY
	Reactions.EmojiName
ORDER BY
	Count DESC
LIMIT ?
OFFSET ?
```

```sql
-- Top 5 Reactions for a User (Scoped to a team incl. DMs/GMs)
SELECT DISTINCT
	EmojiName,
	SUM(EmojiCount) OVER (PARTITION BY EmojiName) AS Count
FROM ((
	SELECT
		EmojiName,
		count(EmojiName) as EmojiCount
	FROM
		Reactions
		INNER JOIN Posts ON Reactions.PostId = Posts.Id
		INNER JOIN Channels ON Posts.ChannelId = Channels.id
	WHERE
		Reactions.DeleteAt = 0
		AND Channels.TeamId = ?
		AND Reactions.UserId = ? 
		AND Reactions.CreateAt > ?
	GROUP BY
			Reactions.EmojiName)
UNION ALL (
	SELECT
		EmojiName,
		count(EmojiName) as EmojiCount
	FROM
		Reactions
		INNER JOIN Posts ON Reactions.PostId = Posts.Id
		INNER JOIN Channels ON Posts.ChannelId = Channels.id
	WHERE
		Reactions.DeleteAt = 0
		AND Reactions.UserId = ?
		AND (Channels.type = 'D' OR Channels.type = 'G')
		AND Reactions.CreateAt > ?
	GROUP BY
		Reactions.EmojiName
)) AS A
ORDER BY
	Count DESC
LIMIT ?
OFFSET ?`
```


## Design Patterns That Could be Used

## Database Schema Changes


## Temporal
- Last 24 hours
- Last 7 days
- Last 30 days

## Scope
- **Team**
	- All public channels and private channels that the user is a member of
- **User**
	- All posts that the user has reacted to within the team and DMs/GMs *or*
	- All posts that the user has reacted to across the workspace (incl. DMs/GMs)

## Authorization
- Does each individual widget require a permission?
- Should have same authorization requirements as top posts/threads

## Guest Visibility
This is up for debate, guests are only granted access to certain channels so should they be able to see reaction info for other channels? The reaction information that is exposed via this widget is minimal so it's up for discussion if it's worth filtering the data further for guest accounts.

## Licensing
- Match overall feature license

## Configs/Feature Flags
- API follows `InsightsEnabled` feature flag.
- This widget will need it’s own config to enable/disable it on the front-end.

## Common Components
- The widget (and its inner elements) will be common components shared across the insights component.
- The bar graph used for the top 5 reactions is shared with the top channels bar graph.

## Security Review

## Telemetry
