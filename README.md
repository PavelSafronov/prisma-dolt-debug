

## nodejs & prisma

install dependencies
`pnpm install`

if prisma is installed globally
`prisma migrate dev`

or if prisma is only installed at project level
`pnpm exec prisma migrate dev`


## dolt

run in an empty dir with database `obsidian` created (`CREATE DATABASE obsidian`)
`dolt sql-server --loglevel=debug`

no config is needed. i used default values in the connection string: `mysql://root@localhost:3306/obsidian`


## note

it works fine with a default my sql install.
dolt also does not work with mikrorm & probably not with typeorm

checked dolt in the past with nocodb (opensource airtable alternative) which worked fine
