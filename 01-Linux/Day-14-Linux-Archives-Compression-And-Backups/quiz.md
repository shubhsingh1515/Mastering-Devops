# Day 14 — Quiz: Linux Archives, Compression & Backups

## Questions

1. **What does `tar -c` mean?**

   <details>
   <summary>Answer</summary>

   Create.
   </details>

2. **What does `tar -x` mean?**

   <details>
   <summary>Answer</summary>

   Extract.
   </details>

3. **What does `tar -t` do?**

   <details>
   <summary>Answer</summary>

   List the contents of an archive.
   </details>

4. **What does the `z` option represent in `tar -czf`?**

   <details>
   <summary>Answer</summary>

   `gzip` compression.
   </details>

5. **What command extracts a `.tar.gz` file?**

   <details>
   <summary>Answer</summary>

   `tar -xzf archive.tar.gz`
   </details>

6. **How do you create a ZIP archive?**

   <details>
   <summary>Answer</summary>

   `zip -r archive.zip directory/`
   </details>

7. **How do you inspect an archive before extraction?**

   <details>
   <summary>Answer</summary>

   `tar -tzf archive.tar.gz` or `unzip -l archive.zip`
   </details>

8. **What is the difference between an application backup and a database backup?**

   <details>
   <summary>Answer</summary>

   An application backup protects the app files and config, while a database backup protects the database state and must be created using database-aware methods.
   </details>

9. **Why should backups be stored away from the production server?**

   <details>
   <summary>Answer</summary>

   A server failure or compromise could destroy both the production system and the local backup.
   </details>

10. **Why must restores be tested?**

    <details>
    <summary>Answer</summary>

    To prove the backup is recoverable and usable in a real incident.
    </details>

---

## Bonus Questions

11. **Why is `tar` often paired with `gzip`?**

    <details>
    <summary>Answer</summary>

    Because `tar` archives files and `gzip` compresses the archive to save space.
    </details>

12. **What does `tar -tzf` do?**

    <details>
    <summary>Answer</summary>

    It shows the contents of a `.tar.gz` archive without extracting it.
    </details>

13. **What is `scp` used for?**

    <details>
    <summary>Answer</summary>

    Securely copying files between machines over SSH.
    </details>

14. **Why should production backups use timestamps?**

    <details>
    <summary>Answer</summary>

    To create unique backup versions and avoid overwriting previous backups.
    </details>

15. **What is one major risk of extracting an untrusted archive as root?**

    <details>
    <summary>Answer</summary>

    It may overwrite critical system files or execute malicious content.
    </details>

---

## Quick Review Answers

- Create
- Extract
- List contents
- `gzip` compression
- `tar -xzf archive.tar.gz`
- `zip -r archive.zip directory/`
- `tar -tzf archive.tar.gz`
- Different recovery mechanisms for different data types
- To protect against local hardware failure or compromise
- To confirm the backup is valid and restorable
