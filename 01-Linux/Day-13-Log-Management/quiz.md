# Day 13 — Quiz: Linux Log Management & Troubleshooting

## Questions

1. **Which directory usually stores Linux logs?**

   <details>
   <summary>Answer</summary>

   `/var/log`
   </details>

2. **Which command follows a log in real time?**

   <details>
   <summary>Answer</summary>

   `tail -f`
   </details>

3. **Which command searches for text?**

   <details>
   <summary>Answer</summary>

   `grep`
   </details>

4. **Which command displays the first lines?**

   <details>
   <summary>Answer</summary>

   `head`
   </details>

5. **Which command displays the last lines?**

   <details>
   <summary>Answer</summary>

   `tail`
   </details>

6. **Which tool views systemd logs?**

   <details>
   <summary>Answer</summary>

   `journalctl`
   </details>

7. **Which log level indicates normal operation?**

   <details>
   <summary>Answer</summary>

   `INFO`
   </details>

8. **Which log level indicates failure?**

   <details>
   <summary>Answer</summary>

   `ERROR` (or `FATAL` for critical failures)
   </details>

9. **Why is log rotation important?**

   <details>
   <summary>Answer</summary>

   To prevent logs from consuming excessive disk space and keep them manageable. Without it, logs fill the disk and cause outages.
   </details>

10. **Why shouldn't secrets appear in logs?**

    <details>
    <summary>Answer</summary>

    Logs may be accessible to others and retained for long periods, creating a serious security and compliance risk.
    </details>

## Bonus Questions

11. **What does `grep -i` do?**

    <details>
    <summary>Answer</summary>

    Performs a **case-insensitive** search, matching both uppercase and lowercase.
    </details>

12. **How do you show the last 20 lines of a log?**

    <details>
    <summary>Answer</summary>

    `tail -n 20 app.log` or `tail -20 app.log`.
    </details>

13. **How do you count the number of ERROR lines in a log?**

    <details>
    <summary>Answer</summary>

    `grep -c "ERROR" app.log`
    </details>

14. **Which two log files are commonly associated with Nginx?**

    <details>
    <summary>Answer</summary>

    `/var/log/nginx/access.log` (requests) and `/var/log/nginx/error.log` (errors).
    </details>

15. **What key in `less` do you press to search?**

    <details>
    <summary>Answer</summary>

    `/` — type the search term, then press `n` to go to the next result and `q` to quit.
    </details>
