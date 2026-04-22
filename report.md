# PES-VCS Lab Report
**Name:** Shristi Saha  
**SRN:** PES1UG24CS903

---

## Phase 1 — Screenshots
- Screenshot 1A: ./test_objects output <img width="805" height="193" alt="1A" src="https://github.com/user-attachments/assets/5580ac5f-aa7c-4f12-9ada-3b2dbbb8f646" />


- Screenshot 1B: find .pes/objects -type f <img width="699" height="138" alt="1B" src="https://github.com/user-attachments/assets/a90fad72-fbdc-4631-bf6c-7fc1fcd3cddc" />


## Phase 2 — Screenshots
- Screenshot 2A: ./test_tree output <img width="1188" height="280" alt="2A" src="https://github.com/user-attachments/assets/0d5d1cd0-e3f1-4146-a765-0293bbae0b75" />

- Screenshot 2B: xxd binary object<img width="1198" height="151" alt="2B" src="https://github.com/user-attachments/assets/c2bcd3c8-3e5d-4bb2-92bf-43cd9b4dba37" />


## Phase 3 — Screenshots
- Screenshot 3A: pes add and pes status output <img width="692" height="552" alt="3A" src="https://github.com/user-attachments/assets/e24e6d26-dd68-4a1f-8ff8-da3518656f2d" />

- Screenshot 3B: cat .pes/index output <img width="780" height="88" alt="3B" src="https://github.com/user-attachments/assets/f0be9a51-9e2f-4dfb-9397-94d69547aeed" />


## Phase 4 — Screenshots
- Screenshot 4A: pes log output <img width="688" height="394" alt="4A" src="https://github.com/user-attachments/assets/5a7bc838-e162-4710-a285-b76c020b96ea" />

- Screenshot 4B: find .pes -type f output <img width="668" height="359" alt="4B" src="https://github.com/user-attachments/assets/fee7c985-47f7-47ab-8fa8-ea52e071e67d" />

- Screenshot 4C: cat .pes/refs/heads/main and HEAD <img width="647" height="106" alt="4C" src="https://github.com/user-attachments/assets/44c5d349-ab05-43e6-99ff-f60d014031cb" />

- Final: Integration test output <img width="635" height="737" alt="final1" src="https://github.com/user-attachments/assets/ebf84cb3-1b34-44cf-94f6-9740ece3d246" />

<img width="810" height="780" alt="final2" src="https://github.com/user-attachments/assets/0f3100b9-2b29-4359-ade5-cb1a1c2fd065" />

---

## Phase 5 — Analysis Questions

### Q5.1 — How would pes checkout work?
Update `.pes/HEAD` to point to the new branch name. Then walk the target commit's tree, retrieve all blobs, and write them to the working directory. If any tracked file has been modified (mtime or size differs from index), refuse checkout and print an error.

### Q5.2 — How to detect dirty working directory?
For each file in the index, compare its stored `mtime_sec` and `size` with the actual file on disk using `stat()`. If they differ, re-hash the file and compare with the stored blob hash. If different, the file is dirty and checkout should refuse.

### Q5.3 — What happens in detached HEAD state?
HEAD contains a commit hash directly instead of a branch reference. New commits are written but no branch pointer is updated. Those commits become unreachable orphans if you switch away. To recover: note the hash from terminal output and run `git branch rescue <hash>`.

---

## Phase 6 — Analysis Questions

### Q6.1 — Garbage collection algorithm?
Start from all branch refs, walk commits to trees to blobs. Collect all reachable hashes into a hash set. Then scan every file under `.pes/objects/` and delete any whose name is not in the set. Use a hash table for O(1) lookups. For 100,000 commits with ~3 objects each, approximately 300,000 objects need to be visited.

### Q6.2 — Race condition between GC and commit?
GC scans reachable objects before a commit writes its new blob. The commit writes the blob, then references it in a tree and commit object. But GC already marked it unreachable and deletes it — leaving a dangling pointer. Git avoids this with a grace period: objects newer than 14 days are never deleted, giving in-flight commits time to complete.
