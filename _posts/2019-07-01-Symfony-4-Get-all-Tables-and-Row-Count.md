---
layout: single
title: Symfony 4 Get all Tables and Row Count
excerpt: "Symfony 4: Get all Tables and Row Count"
date: 2019-07-04
classes: wide
header:
    teaser: /assets/images/allTables.png
    teaser_home_page: true
categories:
- Symfony
tags:
- Symfony
---

I wanted to show all tables from the database and their number of rows in the admin home page. For that, we need to use DBAL.

In our controller:

```php
use Doctrine\DBAL\Driver\Connection;

/**
 * @Route("/admin", name="admin_home", methods={"GET"})
 *
 * @return Response
 */
public function adminindex(Connection $connection): Response
{
    $sql = "SELECT table_name, table_rows from INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA = 'DBNAME';";
    $tables = $connection->fetchAll($sql);

    return $this->render('security/adminindex.html.twig', [
        'tables' => $tables
    ]);
}
```

Now that we have all the data we need it’s time to build our admin_home in twig with this data:
{% raw %}
```php
{% for table in tables %}

    <div class="col-md-3 col-sm-6 col-xs-12">
        <a href="{{ "/" ~ app.request.locale ~ "/admin/" ~ table.table_name }}">
            <div class="info-box">
                <span class="info-box-icon bg-aqua"></span>

                <div class="info-box-content">
                    <span class="info-box-text">{{ table.table_name }}</span>
                    <span class="info-box-number">{{ table.table_rows |number_format(0, ',', '.') }}</span>
                </div>
                <!-- /.info-box-content -->
            </div>
        </a>
        <!-- /.info-box -->
    </div><!-- col-md-3 col-sm-6 col-xs-12 -->

{% endfor %}
```
{% endraw %}