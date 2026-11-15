1\.  \[X] 0001\_initial

&#x20;\[X] 0002\_alter\_servicerequest\_options\_and\_more

&#x20;\[X] 0003\_servicerequest\_resolution\_summary\_and\_more

\--

\-- Change Meta options on servicerequest

\--

\-- (no-op)

\--

\-- Remove index req\_priority\_date\_idx from servicerequest

\--

DROP INDEX `req\_priority\_date\_idx` ON `requests\_app\_servicerequest`;

\--

\-- Add field due\_at to servicerequest

\--

ALTER TABLE `requests\_app\_servicerequest` ADD COLUMN `due\_at` datetime(6) NULL;



2\. | requests\_app\_servicerequest | CREATE TABLE `requests\_app\_servicerequest` (

&#x20; `id` char(32) COLLATE utf8mb4\_unicode\_ci NOT NULL,

&#x20; `title` varchar(160) COLLATE utf8mb4\_unicode\_ci NOT NULL,

&#x20; `description` longtext COLLATE utf8mb4\_unicode\_ci NOT NULL,

&#x20; `location` varchar(160) COLLATE utf8mb4\_unicode\_ci NOT NULL,

&#x20; `priority` smallint unsigned NOT NULL,

&#x20; `status` varchar(20) COLLATE utf8mb4\_unicode\_ci NOT NULL,

&#x20; `solved\_at` datetime(6) DEFAULT NULL,

&#x20; `created\_at` datetime(6) NOT NULL,

&#x20; `updated\_at` datetime(6) NOT NULL,

&#x20; `category\_id` bigint NOT NULL,

&#x20; `requester\_id` int NOT NULL,

&#x20; **`due\_at` datetime(6) DEFAULT NULL,**

&#x20; `resolution\_summary` longtext COLLATE utf8mb4\_unicode\_ci NOT NULL,

&#x20; PRIMARY KEY (`id`),



3\. ... 

{'solved\_at': \['Resolved or closed requests must include a timestamp.'], 'resolution\_summary': \['Resolved or closed requests require a non-empty resolution summary.']}

>>> 



4\. admin.png



5\. >>> queryset = ServiceRequest.objects.filter(

...     status\_\_in=\["open", "assigned"],

...     category\_\_name="Electrical"

... ).order\_by("priority", "-created\_at")

>>> print(queryset.query)

SELECT `requests\_app\_servicerequest`.`id`, `requests\_app\_servicerequest`.`requester\_id`, `requests\_app\_servicerequest`.`category\_id`, `requests\_app\_servicerequest`.`title`, `requests\_app\_servicerequest`.`description`, `requests\_app\_servicerequest`.`location`, `requests\_app\_servicerequest`.`priority`, `requests\_app\_servicerequest`.`status`, `requests\_app\_servicerequest`.`created\_at`, `requests\_app\_servicerequest`.`updated\_at`, `requests\_app\_servicerequest`.`due\_at`, `requests\_app\_servicerequest`.`resolution\_summary`, `requests\_app\_servicerequest`.`solved\_at` FROM `requests\_app\_servicerequest` INNER JOIN `requests\_app\_category` ON (`requests\_app\_servicerequest`.`category\_id` = `requests\_app\_category`.`id`) WHERE (`requests\_app\_category`.`name` = Electrical AND `requests\_app\_servicerequest`.`status` IN (open, assigned)) ORDER BY `requests\_app\_servicerequest`.`priority` ASC, `requests\_app\_servicerequest`.`created\_at` DESC

>>> print(queryset.explain())

\-> Sort row IDs: requests\_app\_servicerequest.priority, requests\_app\_servicerequest.created\_at DESC  (cost=5.01 rows=10)

&#x20;   -> Index range scan on requests\_app\_servicerequest using req\_cat\_status\_idx over (category\_id = 1 AND status = 'assigned') OR (category\_id = 1 AND status = 'open'), with index condition: ((requests\_app\_servicerequest.category\_id = '1') and (requests\_app\_servicerequest.`status` in ('open','assigned')))  (cost=5.01 rows=10)



>>> 



6\. ----------------------------------------------------------------------

Ran 5 tests in 0.157s



OK

Destroying test database for alias 'default' ('test\_sen305\_servicedesk')...

(.venv) Nita@Nita:\~/sen305-service-platform/backend$ 



7\. Data safety; Only synthetic laboratory users (requester1, technician1) and auto-generated test records were used during execution; no production or sensitive data was accessed or modified.

