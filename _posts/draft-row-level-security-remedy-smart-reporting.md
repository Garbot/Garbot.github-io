Prior to version 9.1, BMC Remedy reporting was a bit of a hassle.  Let's say you had 10 different people within your organization that needed a report of all their outstanding work orders with the basic info:

* Work Order ID
* Assignee
* Status

You'd have to create 10 separate reports.  You can now take advantage of the row level security feature in Smart Reporting to minimize the amount of work needed.

```SQL
SELECT 'User ID',
`CTM:SupportGroupAssocPeopleLookUp`.`Login ID`+'@bmc',
'Support Group',
`CTM:Support Group`.`Support Group Name`
FROM `AR System Schema`.`CTM:SupportGroupAssocPeopleLookUp`
INNER JOIN `AR System Schema`.`CTM:Support Group`
ON
( `CTM:SupportGroupAssocPeopleLookUp`.`Support Group ID` = `CTM:Support Group`.`Support Group ID`
)
```

Here's another useful example, where we filter based on Site.  Using this filter, users will only be able to view tickets corresponding to their office location.

```SQL
SELECT 'User ID' as IdentifierType,
`CTM:People`.`Remedy Login ID`+'@bmc',
'Site Name' as FilterType,
`SIT:Site`.`Site`
FROM `AR System Schema`.`CTM:People`
INNER JOIN `AR System Schema`.`SIT:Site`
ON
( `CTM:People`.`Site` = `SIT:Site`.`Site`)
```
