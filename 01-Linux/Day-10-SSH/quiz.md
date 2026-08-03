# Day 10 — Quiz: SSH (Secure Shell)

## Questions

1. **What does SSH stand for?**

   <details>
   <summary>Answer</summary>

   **Secure Shell** — A protocol for secure, encrypted remote access and management of computers over a network.
   </details>

2. **What is the default SSH port?**

   <details>
   <summary>Answer</summary>

   **22** (TCP). This is the default port SSH listens on and connects to.
   </details>

3. **Which command generates an Ed25519 SSH key?**

   <details>
   <summary>Answer</summary>

   `ssh-keygen -t ed25519` — Generates a modern, secure Ed25519 key pair.
   </details>

4. **Which file stores authorized public keys on the server?**

   <details>
   <summary>Answer</summary>

   `~/.ssh/authorized_keys` — This server-side file contains the public keys of users allowed to log in.
   </details>

5. **Which file stores known server fingerprints on the client?**

   <details>
   <summary>Answer</summary>

   `~/.ssh/known_hosts` — This client-side file stores the fingerprints of servers I've connected to before, helping detect identity changes.
   </details>

6. **Which command copies files over SSH?**

   <details>
   <summary>Answer</summary>

   `scp` (Secure Copy) — For example, `scp app.js ubuntu@203.0.113.10:/home/ubuntu/`.
   </details>

7. **What permission should a private key have?**

   <details>
   <summary>Answer</summary>

   **600** (rw-------) — Only the owner can read/write. This prevents other users from reading the secret key.
   </details>

8. **Which command adds a key to the SSH agent?**

   <details>
   <summary>Answer</summary>

   `ssh-add ~/.ssh/id_ed25519` — Adds the decrypted key to the running SSH agent.
   </details>

9. **Why should you never share a private key?**

   <details>
   <summary>Answer</summary>

   The private key grants access to any system that trusts the corresponding public key. Sharing it means anyone holding it can impersonate me and access those systems.
   </details>

10. **Why is key-based authentication preferred?**

    <details>
    <summary>Answer</summary>

    It's more secure than password-based authentication (no password travels over the network) and better suited for automation and scripting.
    </details>

## Bonus Questions

11. **What is the difference between a public key and a private key?**

    <details>
    <summary>Answer</summary>

    The **private key** stays on my computer and is never shared; it proves my identity. The **public key** is safe to share and is placed on servers to verify my identity. Together they form a key pair.
    </details>

12. **What is the purpose of the SSH config file?**

    <details>
    <summary>Answer</summary>

    `~/.ssh/config` lets me define aliases for servers with their hostname, username, and key. For example, I can type `ssh prod-api` instead of a long command.
    </details>

13. **What is the role of the SSH agent?**

    <details>
    <summary>Answer</summary>

    The SSH agent keeps decrypted SSH keys in memory so I don't have to enter my passphrase repeatedly. It's useful for automation and multi-hop connections.
    </details>

14. **Why is SSH preferred over Telnet?**

    <details>
    <summary>Answer</summary>

    SSH encrypts all traffic (including passwords and commands), while Telnet sends everything in plaintext that anyone on the network can read.
    </details>

15. **What is the difference between `authorized_keys` and `known_hosts`?**

    <details>
    <summary>Answer</summary>

    `authorized_keys` is on the **server** and lists public keys allowed to log in (server trusts clients). `known_hosts` is on the **client** and stores server fingerprints (client trusts servers).
    </details>
</content>
