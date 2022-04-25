# Tech Spec: Top Boards, My Top Boards

## Designs

See iteration 6 in the [Figma design](https://www.figma.com/file/fKyLulu6MUjQzQa9pBFFrC/Insights-and-Analytics?node-id=548%3A102444).

## Components 

1. Board icon or default dartboard icon if the board has no custom icon
1. Board name or `(Untitled Board)` if the name hasn't been set
1. Count of activity as found in the new `focalboard_boards_history` and `focalboard_blocks_history` tables
1. Avatars of participants
    * a "participant" is anyone who is active in the given time range (1, 7, or 28 days)
    * "active" means they were the `modified_by` user of a record in `focalboard_boards_history` or `focalboard_blocks_history`
    * include the creator of the board in the participants list even if they were not active
1. My boards
    * A board is "mine" if I was active (see above for definition) during the given period or if I was the creator

## SQL

Something like this (not optimized):

* count the records in the `boards_history` and `blocks_history` tables:
    * for the given team 
    * for the given time range
    * excluding deleted boards
    * excluding records created by the system

```sql
select
	id,
	title,
	sum(count),
	string_agg(distinct modified_by, ',') as users
from
	(
		select
			boards.id,
			boards.title,
			count(boards_history.id) as count,
			boards_history.modified_by
		from
			focalboard_boards_history as boards_history
			join focalboard_boards as boards on boards_history.id = boards.id
		where
			boards_history.insert_at > now() - interval '28 day'
			and boards.team_id = '9pimkfeqi38w5jq4wxpiss98sr'
			and boards_history.modified_by != 'system'
			and boards.delete_at = 0 
			-- and (boards.created_by = '58wh73bt1inkdbnzyjciboe8ic' or boards_history.modified_by = '58wh73bt1inkdbnzyjciboe8ic')
		group by
			boards_history.id,
			boards.id,
			boards_history.modified_by
		UNION
		ALL
		select
			boards.id,
			boards.title,
			count(blocks_history.id) as count,
			blocks_history.modified_by
		from
			focalboard_blocks_history as blocks_history
			join focalboard_boards as boards on blocks_history.board_id = boards.id
		where
			blocks_history.insert_at > now() - interval '28 day'
			and boards.team_id = '9pimkfeqi38w5jq4wxpiss98sr'
			and blocks_history.modified_by != 'system'
			and boards.delete_at = 0
			-- and (boards.created_by = '58wh73bt1inkdbnzyjciboe8ic' or blocks_history.modified_by = '58wh73bt1inkdbnzyjciboe8ic')
		group by
			blocks_history.board_id,
			boards.id,
			blocks_history.modified_by
	) as boards_and_blocks_history
group by
	id,
	title
limit
	4;
```