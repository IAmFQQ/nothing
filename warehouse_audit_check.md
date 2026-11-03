---

**Warehouse Audit Check**

Each time a shipment arrives at a warehouse, the system records the part codes in the shipment as a string array `parts`. The quality team keeps its own list of part codes that require inspection, given as a string array `audit`.

A shipment must be held for inspection when **two or more entries** in `parts` are identical to some code in `audit`. Implement `needsAudit(parts, audit)`. It returns `true` if the shipment must be held and `false` otherwise.

Matching rules:

- An entry counts only if the whole string is equal to a code in `audit`. Partial matches, prefixes, and substrings do not count.
- Every position in `parts` is counted separately. If the same code appears twice in `parts`, it counts twice.
- Repeated codes in `audit` have no extra effect.
- The matching entries do not need to be adjacent, and their order does not matter.

**Example 1**

```
Input:  parts = ["cedar","linen","harbor"], audit = ["harbor","cedar"]
Output: true
```

`"cedar"` and `"harbor"` are both on the audit list. That makes two matching entries.

**Example 2**

```
Input:  parts = ["cedar","violet","kettle"], audit = ["harbor","violet","mosaic"]
Output: false
```

Only `"violet"` matches, so there is just one matching entry.

**Example 3**

```
Input:  parts = ["linen","linen"], audit = ["linen"]
Output: true
```

The same code fills two positions, so it counts as two matching entries.

**Example 4**

```
Input:  parts = ["harbor","harboring"], audit = ["harbor"]
Output: false
```

`"harboring"` is not equal to `"harbor"`, so only one entry matches.

**Constraints**

- `1 <= parts.length, audit.length <= 10^5`
- `1 <= parts[i].length, audit[i].length <= 15`
- All strings consist only of lowercase English letters.



**Solution**

```
class Solution {
    public boolean needsAudit(String[] parts, String[] audit) {
        
    }
}
```
