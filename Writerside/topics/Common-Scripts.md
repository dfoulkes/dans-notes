# Common Scripts


## Extracting Story Points from Jira Database


This query can be used to find stories that have been completed and for the year 2019
and list the story points, resolution date, created date and the project name.
```sql
SELECT CONCAT(pj.pkey, '-', + ji.issuenum)  as ticket, ji.DESCRIPTION, ji.SUMMARY, cf.cfname, cfv.NUMBERVALUE, ji.RESOLUTIONDATE, ji.CREATED, pj.pname
FROM jiraissue ji
JOIN customfieldvalue cfv ON ji.ID = cfv.ISSUE
JOIN customfield cf ON cfv.CUSTOMFIELD = cf.ID
join project pj on pj.id = ji.PROJECT
where year(ji.RESOLUTIONDATE) = 2019
and cf.cfname like 'Points' 
```


