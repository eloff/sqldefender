---
type: faq
breadcrumb: FAQ
subtitle: Frequently Asked Questions
sections:
  - general
  - performance & scaling
  - billing
  - audit logs
  - compatibility
  - how do i?
faq:
  general:
    - question: Is SQL Defender open-source?
      answer: "It’s source available, but not OSI open-source. We want to produce an invaluable product for our users
        and a successful business. A successful business in turn provides the funding and support to keep improving 
        the product. The two are inseparable and we’d be doing ourselves and our users a disservice by choosing a
        software license that hobbles either. It's all about choosing a sustainable path for the future. We love and
        use and contribute to open-source software, but it’s well known that it’s hard to monetize effectively. This
        is exacerbated by the current trend of massive cloud companies monetizing open-source products by providing
        them as a service without contributing back. There are a few projects like Linux that are notable exceptions,
        but by and large they’re the exceptions to the rule. For every one success there are tens of thousands of
        projects that languish neglected in obscurity because the creators can’t quit their day job. The difficulty in monetizing open-source has deprived the software ecosystem of better maintained and higher quality software
        and we're all poorer for it. We still provide the source code to ensure you can read it to cover
        short-comings in the documentation, fix bugs, or make changes for your specific use case.

        #### What about the (A)GPL?

        It's a radioactive license to large organizations, that makes it a non-starter."
    - question: Why not use Postgres RLS?
      answer: "The most common reaction lot of engineers hearing about SQL Defender is “why not just use row-level
        security (RLS) in PostgreSQL?” It’s a fair question, and we initially considered RLS very seriously for this task, but decided upon further examination that it wasn’t a good fit. So SQL Defender was born. The reasoning would make a better blog post than a faq entry, but here's a summary of the argument.

        #### Performance

        To prevent leaking information, RLS filters are materialized before joins or where clause filtering which can
        turn every query into a table scan unless you’re careful to index every RLS condition and mark all functions
        involved as leakproof. That can make it difficult to use right for unsuspecting developers - and we
        specifically want SQL Defender to be simple and obvious enough for junior developers to use efficiently and
        correctly.

        #### Not fully-supported on RDS

        In order to solve the performance problems above and let the Postgres optimizer execute the query in the
        most efficient way possible, you need to mark any functions used as leakproof. That requires superuser
        privileges not available on Amazon’s RDS. This means you have to choose between RDS and efficient RLS
        queries. RLS is a nice to have, but RDS is often a non-negotiable requirement.

        #### Overlapping, but not identical requirements

        RLS is great, and has some overlapping requirements with SQL Defender, but it also has subtly different
        objectives. For example the need to protect against user defined functions leaking secure information
        is just not a design concern with SQL Defender. We’re not trying to allow untrusted users to create and
        use user-defined functions safely. We also want to be able to execute query data on behalf of multiple 
        users safely. Database users is not really the right level of granularity to map one-to-one with application
        users. Eventually these little mismatches would mean greater total complexity if we built of RDS instead of
        using our own purpose built system.

        #### No audit log

        We need a full, searchable, tamper-resistant audit log of every interaction with the database, tied to the
        application or organization user who initiated it. This is sometimes a regulatory requirement, but it’s
        also just good opsec. This requirement is out scope for RLS.

        #### RLS is PostgreSQL only

        SQL Defender is PostgreSQL first, but it’s meant to be ported to other RDBMS systems, and relying so
        heavily on PostgreSQL only features for core functionality would mean we’d need a different and
        incompatible approach for other databases, at which point we have two systems to maintain."
  compatibility:
    - question: Does SQL Defender work with Amazon RDS, Aurora, or Google Cloud SQL?
      answer: Yes, these are fully supported. In fact SQL Defender can help with common problems like the low
        connection limits on RDS that people frequently run into trouble with, thanks to connection pooling.
        For the same reason it can enable serverless architecture using Lambda or Cloud Functions where database
        connection limits are a limiting factor.
    - question: Does SQL Defender work with other PostgreSQL compatible databases like CockroachDB or YugaByte DB?
      answer: We only officially test with PostgreSQL currently, but depending how compatible the database is
        it may work with SQL Defender. Give it a try and contact support if you run into problems. Also
        please share your experiences with the community so we can have a better answer here!
    - question: What about load balancing or failover between multiple PostgreSQL databases?
      answer: This is not supported yet, but you can accomplish this by putting SQL Defender in front
        of pgpool or pgbouncer.
  performance & scaling:
    - question: How does caching work?
      answer: You can enable query caching in SQL Defender which will use the incoming mutation queries 
        (updates, inserts,  delete, etc) to invalidate cached results. Currently caching is limited to very simple
        kinds of queries, but that   are nonetheless common. Specifically we also cache user roles to speedup security
        rules in queries. This kind of cache invalidation only works if all database connections go through SQL Defender.
        If updates can be run directly against the database then the cache can get out of sync. Caching can be disabled
        in that case, but the smarter advice would just be "if it hurts, don’t do it".
    - question: How well does SQL Defender scale?
      answer: Typically better than PostgreSQL scales, so it's rare for SQL Defender to become the bottleneck.
        SQL Defender is fully multi-core and scales well onto larger machines with more cores and more memory.
        We’re working on adding more advanced caching and optimizations to alleviate load on the database and 
        scale to higher levels. We also want to enable horizontal scaling, both for SQL Defender and the backing
        database in the future.
        # When the database becomes overwhelmed, SQL Defender protects it by introducing back pressure, rejecting the
        # most expensive queries. This keeps your database as available as possible when something has to give.
    - question: How do access rules affect performance?
      answer: Incoming queries are parsed, transformed by adding in the security rules, and sent to the database as
        a single unified query. Additional joins and calling custom functions may add some time to the queries, and
        performance will vary with the precise rules defined. Because no additional round-trips to the database are
        required, the impact can be quite minimal. SQL Defender does connection pooling and offers optional 
        deduplication and caching of queries. Putting SQL Defender in front of your database can seriously improve
        performance, depending on access patterns, and it’s only going to get better with time.
  how do i?:
    - question: What types of queries can I target with access rules?
      answer: Targets can be * for all queries, or select, update, insert, delete, or read=select, write=update,
        insert, and delete. There is also ddl which grants access to create, drop, and alter statements. Currently
        DDL statement rules are limited to binary allowed (true) or not allowed (false). It's meant for
        administrator/developer roles. Truncate is allowed only if delete is unconditionally allowed (true).
        Upsert is not yet supported, but coming with high priority.
    - question: What is strict mode?
      answer: In order to strike a balance between ease of development and security, strict mode is disabled by
        default. Without strict mode, security rules are applied only to the top-level select clause and columns
        and tables being returned from the query. However, this still permits side-channel attacks from user-generated
        queries by joining to rows that are not allowed to be accessed and then toggling returned data based on these
        hidden values. For example the query
        ```SELECT 1 FROM allowed INNER JOIN hidden ON hidden.allowed_id = allowed.id WHERE hidden.amount > 100;```  leaks information if a prviate column has an amount greater than a given threshold. In general this is a
        reasonable tradeoff to make, because you don’t write application queries that leak information like this.
        But if you allow untrusted queries, e.g. some form of user-defined queries, then you want stricter access 
        control closer to what PostgreSQL RLS provides where access controls are applied to all tables in the query, 
        not just the returned columns. Strict mode checks all tables in the query and prevents these kinds of leaks,
        it's closer to how PostgreSQL RLS works.
    - question: How do roles work?
      answer: Roles are stored in tables created by SQL Defender. Roles behave much as you’d expect, a user can have
        many roles, and each role can be used to grant row, column, and sort permissions, etc.
        If a user has more than one role involved in a security rule, they get access if any of the rules pass 
        (rules are ORed together.) Roles are also hierarchical, so you can group roles into arbitrary hierarchies
        and the user acts as if they’re a member of all roles directly assigned to them, plus all roles that
        descendants of those roles. For example an admin role might be a superset of a supervisor role, itself a
        superset of the staff role, etc. There’s also the special public role which represents permissions granted
        to unauthenticated users. Access rules can target one or more roles as well as authenticated (any role) or
        public (applied to all connections.) You create roles and assign them to your users with standard SQL queries according to your application logic.
    - question: What about ABAC (attribute-based access control)?
      answer: The security rules system is a superset of an ABAC system. Access can be controlled by roles, by
        attributes on the user, attributes on the rows themselves, or attributes in other tables linked through
        relational joins. Given that you have a full turing complete programming language available in the form of
        database functions and procedures, e.g. through PL/SQL, you can theoretically use any access control
        paradigm, or mix and match however you please.
    - question: What about capability based security?
      answer: Capability-based security is another security paradigm where possession of a capability both
        designates the resources and grants some kind of permissions to access that resource. Security is
        achieved by controlling possession. As with ABAC, you have full freedom to structure your security
        rules as you like with all of the power of PostgreSQL at your disposal, so there is nothing preventing
        you from using a capability based access model.
    - question: What about very complex domain security logic?
      answer: What if you want to grant read access to a few specific columns, write access to different columns,
        and only to manager roles, where their email address ends in acme.com, only on Mondays, and only to rows
        resources where the created_by field references users which are related to the manager as subordinates?
        You can implement that with SQL Defender. In addition to the flexible and powerful column and row based
        security, you can write and call custom PL/SQL functions to your heart’s content. With all the power of
        PostgreSQL at your disposal, you can implement the most complex of access rules, while keeping all of
        the logic together in one place, separate from the rest of your code, and easy to audit for correctness.
        # In a future release we want to add further ability to customize all phases of the application of 
        # security rules and database queries using javascript middleware.
  billing:
    - question: Do you offer free licenses for non-commerical use?
      answer: Yes, contact us and tell us about your project and we'll set you up with a free non-commercial license.
        Some features aren't included.
    - question: Do you have a free trial or a startup version? 
      answer: We have a free tier with no time limits, we bill based on the volume of database queries. Some features
        aren't included. If for some  reason you cannot afford the license, contact us so we can understand your
        situation better and see if we can offer a solution.
  audit logs:
    - question: How do audit logs work?
      answer: All connections to the database go through SQL Defender, whether they be from your application, your
        background tasks, dashboards, analytics tools, or developers. We store a tamper-proof log of all actions taken
        by all users on the database, along with the current time, a snapshot of their roles, and any custom audit data
        that you choose to include. This log utilizes similar blockchain technology made famous by Bitcoin, ensuring
        that the entire audit history has been verifiably not tampered with.
    - question: What if I just want the audit logs?
      answer: "You can set a single access rule for all your tables using `table: *` that grants all read/write 
        access. That effectively makes the security layer a pass-through, but you’ll still have the benefit of the
        audit logs. We strongly recommend defining more detailed security rules even if it only encapsulates part of
        the access control logic, and the rest remains in the application. This follows a defense in depth philosophy
        where you don’t rely on a single layer for protection."
---