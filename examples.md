```sh
/usr/local/Cellar/sqlite/3.37.2/bin/sqlite3
```

```sql
-- setup
create table talks(
  talks_id integer primary key,
  title,
  description
);
create table users(
  user_id integer primary key,
  name,
  most_recent_talk_id
);

-- create user
insert into
  users(name)
values
  ('Emil');

select * from talks;
select * from users;

-- create 1
insert into
  talks(title, description)
values
  ('something else', 'ramblings');

-- create 2
insert into
  talks(title, description)
values
  ('sql notes', 'ramblings')
returning
  talks_id;

-- update
update users
  set most_recent_talk_id = '???'
where name = 'Emil';

select * from users;


-- drop table talks;
-- drop table users;
```
