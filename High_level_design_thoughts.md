## High-level thoughts

[edited 20140319; stopped work 11:48 and began work at https://github.com/brannerchinese/regex_engine_py/blob/master/design.md]

### Idea 1: NFA, implemented as struct and stack

1. **Struct**: The search pattern is incrementally expanded into a verbose series of elements in a struct (C) or list (Python). With each successive change, the newest element is compared with the next element to be sought in the target.

  2. Use switch-statements (C) or hash-table-branching (Python) to assign expansion-functions to regex operators.
  2. Parenthesis-bounded groups, since they may themselves contain operators, are best treated as nested structs/lists.
  2. [here, need to detail how each operator is treated]

1. **Stack**: Incremental matches in the target string are pushed one after another onto a stack. 

  2. When a complete match is achieved, the procedure is finished. 
  2. If a complete match fails to be achieved, the partial match found so far is abandoned and a new version of the stack is created, starting from the next match.
  2. Advantage: wildcard `.` can be checked very fast, since the cardinality rather than the content is to be matched.
  2. Problem: how to reuse matching content within an abandoned partial match. (An efficiency to be considered later.)

---

### Idea 2: Find consecutive sequence of all found elements

1. Expand search pattern as before.
1. For each distinct element in the search pattern, make a list of its appearances by index in the target string.
1. Sort each list of indices and then find the first consecutive sequence of these indices. 
1. Problems: 

  2. Variable-length and optional-content items could lead to huge numbers of items to be treated. But how does this compare to the first method? Is it possible to decide beforehand which is more efficient, by counting the number of `.` and negated character classes, and then use whichever seems faster? 
  2. This method avoids all time wasted backtracking but makes numerous unnecessary comparisons.
  3. Wildcard `.` and large character classes will make the number of search strings huge.

---

### Idea 3: Compare all search strings against subsets of target

1. Create a collection of all possible matching strings, sort by length (long to short), and compare to all substrings of that size within the target.
1. Problem: wildcard `.` and large character classes will make the number of search strings huge.
1. Is the sorting an important savings? Perhaps can save time with generator expression.

---

### Idea 4: Diminishing index of plausible next steps

1. Traverse target once and make "candidate" list of indices bearing elements that match the first element of the search string.
1. For each item in candidate list, compare `target[item+1]` against the second element of the search string. Prune candidate list accordingly.
1. Continue until search string is exhausted or candidate list is empty.
1. Problem: each optional-content item still creates multiple lists that have to be handled.

[end]