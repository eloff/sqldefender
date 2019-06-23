---
breadcrumb: Home
subtitle: Frequently Asked Questions
sectionOrder:
  - General
  - Performance & Scaling
  - Billing
  - Audit Logs
  - Compatibility
  - Howto
  
faq:
  - General:
      - question: Is SQL Defender open-source?
        answer: It’s source available, but not OSI open-source. This is a complicated issue so we'll only summarize the
          argument here. The folks at TimescaleDB made a similar change for much the same reasons, and their logic
          is [well explained](https://blog.timescale.com/how-we-are-building-an-open-source-business-a7701516a480/).
          The gist is we want to produce an invaluable product for our users and a successful business. A successful
          business in turn provides the funding and support to keep improving the product. The two are inseparable and
          we’d be doing ourselves and our users a disservice by choosing a software license that hobbles either. It's all
          about choosing a sustainable path for the future. We love and use and contribute to open-source software, but
          it’s well known that it’s hard to monetize effectively. This is exacerbated by the current trend of massive
          cloud companies monetizing open-source products by providing them as a service without contributing back.
          There are a few projects like Linux that are notable exceptions, but by and large they’re the exceptions to
          the rule. For every one success like that there are tens of thousands of projects that languish in obscurity because the creators can’t quit their day job.
      - question: 
        answer:
      - question: Do you have a managed service?
        answer: It’s on the roadmap, but currently you’ll need to host it yourself. We try to make this process as 
          easy as possible by supplying a single binary with no dependencies, and docker and cloud provider images.
          Please contact us if you have difficulty operating SQL Defender.
  - Compatibility:
      - question: Does SQL Defender work with Amazon RDS, Aurora, or Google Cloud SQL?
        answer: Yes, these are fully supported. In fact SQL Defender can help with common problems like the low
          connection limits on RDS that people frequently run into trouble with, thanks to connection pooling.
          For the same reason it can enable serverless architecture using Lambda or Cloud Functions where database
          connection limits are a limiting factor.
      - question: Does SQL Defender work with other PostgreSQL compatible databases like CockroachDB or YugaByte?
        answer: We only officially test with PostgreSQL currently, but depending how compatible the database is
        it may work with SQL Defender. Give it a try and contact support if you run into problems. Also
        please share your experiences with the community so we can have a better answer here!
      - question: What about load balancing or failover between multiple PostgreSQL databases?
        answer: This is not supported yet, but you can accomplish this by putting SQL Defender in front
          of pgpool or pgbouncer.
  - Performance & Scaling:
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
  - Howto:
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
  - Billing:
      - question: Do you offer free licenses for non-commerical use?
        answer: Yes, contact us and tell us about your project and we'll set you up with a free license.
      - question: Do you have a free trial or a startup version? 
        answer: No, we have a free tier. All features, except for enterprise features are available to all tiers,
          we bill based on the number of database queries. If for some reason you cannot afford the license,
          contact us so we can understand your situation better and see if we can offer a solution.
  - Audit Logs:
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