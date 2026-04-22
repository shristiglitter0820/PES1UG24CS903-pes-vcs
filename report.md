# PES-VCS Lab Report
**Name:** Shristi Saha  
**SRN:** PES1UG24CS903

---

## Phase 1 — Screenshots
- Screenshot 1A: ./test_objects output (paste screenshot)
- Screenshot 1B: find .pes/objects -type f (paste screenshot)

## Phase 2 — Screenshots
- Screenshot 2A: ./test_tree output (paste screenshot)
- Screenshot 2B: xxd binary object (paste screenshot)

## Phase 3 — Screenshots
- Screenshot 3A: pes add and pes status output (paste screenshot)
- Screenshot 3B: cat .pes/index output (paste screenshot)

## Phase 4 — Screenshots
- Screenshot 4A: pes log output (paste screenshot)
- Screenshot 4B: find .pes -type f output (paste screenshot)
- Screenshot 4C: cat .pes/refs/heads/main and HEAD (paste screenshot)
- Final: Integration test output (paste screenshot)

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
