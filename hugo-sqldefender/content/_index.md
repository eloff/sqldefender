---
breadcrumb: Home
subtitle: Access control proxy with audit logs for PostgreSQL
blurbs:
  secure:
    title: Secure your app by reducing trusted surface area
    content: "![Security](images/yellow-lock-icon.png) Keep your security rules together in one place
      using a common permissions framework. This makes them easy to reason about, facilitates regular
      audits, and _reduces serious data-leak bugs_. A typical application has the security access rules
      scattered throughout the code via various disjoint mechanisms. It's technical debt that grows ever
      harder to reason about, and makes the trusted computing base (TCB) include the entire application.
      With SQL Defender, the TCB shrinks to just include	your access logic plus your database."
  performance:
    title: Improve PostgreSQL Performance
    content: "![PostgreSQL](images/PostgreSQL.svg) Access control rules are optimized and compiled into the query.
      The planner is free to execute that query with its joins in the optimal order, [unlike with PostgreSQL's RLS](/faq/#RLS) restrictions. Connection pooling reduces load and memory usage on the database server.
      Query deduplicaton and in-memory caching reduce load on the database and improve query latency."
      #Back-pressure rejects the most expensive queries when the database is overloaded.
  simple:
    title: Make the simple easy, and the hard possible
    content: Common tasks of restricting access to a user's own data, or by read / insert / update / delete, 
      or restricing visible columns are	easy. With the ability to pass arbitrary context from the
      application and write turing complete PL/SQL functions to control access on a row-level,
      you can codify the most complex domain logic. Keep all your rules, including PL/SQL in your version
      control and deploy with a single command. SQL Defender will handle installing them to the database.
  injection:
    title: Stop SQL injection attacks
    content: "![Stop Hackers](images/stop-hackers.jpg) Because SQL injection happens at the untrusted 
      application layer, SQL Defender rewrites or rejects those queries	the same as any others to enforce
      the security access rules. Multiple queries delimited with semicolons are not permitted,
      shutting down a whole class of SQL injection attacks. Any compromise of the application layer
      won't lead to unauthorized database access, because the application isn't inside the trusted base."
  audit:  
    title: Blockchain audit logs saved to S3
    content: "![Audit Logs](images/audit.svg) Every block appended to the audit log
      forms part of a cyptographic chain with the blocks that come before it, similar to
      how it works in Bitcoin. This makes certain that every operation was performed as
      logged without any retroactive changes. Altering any part of the log invalidates
      all of the subsequent block signatures making the tampering evident.
      The logs are encrypted and saved to	S3 compatible cloud storage or an
      on-premises version like Minio. The encrypted logs are replicated to our 
      offsite cloud data center and automatically verified that the copies match when
      reading the logs back, ensuring they've not been modified in one location.
      You can create filters to keep sensitive information out of your logs."

---