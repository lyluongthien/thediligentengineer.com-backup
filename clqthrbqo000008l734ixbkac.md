---
title: "Write a find-and-replace-pattern LeetCode solution that beats 100.00% of users with TypeScript [54ms]"
datePublished: Sun Dec 31 2023 12:52:14 GMT+0000 (Coordinated Universal Time)
cuid: clqthrbqo000008l734ixbkac
slug: write-a-find-and-replace-pattern-leetcode-solution-that-beats-10000-of-users-with-typescript-54ms
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1704027081631/29b486d8-5aaa-4d78-ae60-3465846364cf.png
tags: typescript, leetcode

---

> Try to solve the solution before read this article at: [leetcode.com/problems/find-and-replace-pattern](https://leetcode.com/problems/find-and-replace-pattern)
# Intuition
<!-- Describe your first thoughts on how to solve this problem. -->
**Thoughts**:
- Focus on matching patterns, not exact characters. The problem doesn't require matching exact characters, but rather identifying words that follow the same pattern as the given pattern string.
- Character consistency is crucial. A word matches the pattern if we can consistently replace each character in the pattern with a unique character in the word, and vice versa.


# Approach
<!-- Describe your approach to solving the problem. -->
1.  Transform Words and Pattern:

    -   Create a function `wordToId` that transforms a word into a pattern-like string:
        -   It assigns a unique integer ID to each distinct character in the word.
        -   It joins these IDs using hyphens to create a pattern-like representation.
    -   Apply `wordToId` to both the `pattern` and each word in the `words` array.
2.  Identify Matching Words:

    -   Iterate through each word in the `words` array.
    -   If the transformed pattern-like string of the word matches the transformed `pattern`, add the original word to the `result` array.
3.  Return Matching Words:

    -   Return the `result` array containing the words that match the pattern.

# Complexity
-   Time complexity: O(n * m), where n is the number of words in the `words` array and m is the length of each word (and the `pattern`). This is due to iterating through each word and transforming it character by character.
-   Space complexity: O(m), where m is the length of the words and pattern. The space used is primarily for the hash map in `wordToId` and the transformed strings.

# Code

```typescript
function findAndReplacePattern(words: string[], pattern: string): string[] {
  const result: string[] = [];
  const wordToId = ((w: string) => {
    const idGetter = {
      hMap: {} as Record<string, number>,
      id: 0,
      getId() {
        return ++this.id
      },
      reset() {
        this.id = 0
        this.hMap = {}
      }
    }
    return [...w].map(c => {
      if (idGetter.hMap[c]) {
        return idGetter.hMap[c]
      } else {
        idGetter.hMap[c] = idGetter.getId()
        return idGetter.hMap[c]
      }
    }).join('-')
  })
  const p = wordToId(pattern)
//   console.log({ p })
  words.forEach(w => {
    if (w.length !== pattern.length) return
    const id = wordToId(w) 
    if (p === id) {
      result.push(w)
    }
  })

  return result
};
// console.log(findAndReplacePattern(words, pattern))
```

# Reference
The LeetCode submission: [leetcode.com/problems/find-and-replace-pattern/submissions/1133020282](https://leetcode.com/problems/find-and-replace-pattern/submissions/1133020282/)
