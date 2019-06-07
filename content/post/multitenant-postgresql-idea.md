---
title: "Multitenant Postgresql Idea"
date: 2019-06-07T12:43:29+02:00
draft: true
---
I recently migrate a postgresql DB from an ubuntu server VM to Azure Postgresql. I you use this
DBaaS before, you may have noticed you need to add `@<my_instance_name>` after you're login. For
exemple if you create a `test_azure_db`, you're url for this postgresql instance is
`test_azure_db.postgres.database.azure.com`. And imagine the name of your user is `remi`, to login
you need to do `psql -h test_azure_db.postgres.database.azure.com -U remi@test_azure`.



