---
name: hello-world
description: Advanced hello world test for AI Program VM. Stresses recursion/delegation with nested tree structures, mixed parallel/sequential dispatch, and multi-level aggregation.
version: 3.0.0
requires:
  tools: [write]
  skills: [ai-program-interpreter]
---

<!-- EXECUTE THIS PROTOCOL IMMEDIATELY WHEN READ -->

```basic
SKILL hello-world WITH name, output_path, names, output_dir, parallel, depth, branch_factor, nested_mode, tree_root

  # Set defaults for single mode
  SET default_name = "World"
  SET default_output = "hello.txt"
  SET default_parallel = false
  SET default_depth = 2
  SET default_branch_factor = 3
  SET default_nested_mode = false

  # Nested mode: create recursive tree of delegations
  IF nested_mode = true THEN
    GOTO nested_dispatch
  END IF

  # Tree root mode: entry point for nested tree
  IF tree_root IS DEFINED AND output_dir IS DEFINED THEN
    GOTO tree_dispatch
  END IF

  # Multi-mode requires names + output_dir
  IF names IS DEFINED AND output_dir IS DEFINED THEN
    GOTO multi_mode
  END IF

  # Single mode: use provided values or defaults
  IF name IS NOT DEFINED THEN
    SET name = default_name
  END IF

  IF output_path IS NOT DEFINED THEN
    SET output_path = default_output
  END IF

  # Generate greeting message
  SET greeting = "Hello, " + name + "!"

  # Write greeting to file
  CALL tool "write" WITH path = output_path, content = greeting

  # Return success with results
  RETURN status = "success", message = greeting, path = output_path

nested_dispatch:
  # Set defaults for nested mode
  IF depth IS NOT DEFINED THEN
    SET depth = default_depth
  END IF

  IF branch_factor IS NOT DEFINED THEN
    SET branch_factor = default_branch_factor
  END IF

  IF parallel IS NOT DEFINED THEN
    SET parallel = default_parallel
  END IF

  IF output_dir IS NOT DEFINED THEN
    SET output_dir = "./nested_out"
  END IF

  # Build the tree from root
  SET root_result = FOLLOW skill "hello-world" WITH tree_root = "Root_" + depth, output_dir = output_dir, depth = depth, branch_factor = branch_factor, parallel = parallel, current_depth = 0

  RETURN status = "nested_complete", tree_result = root_result

multi_mode:
  # Set parallel flag default
  IF parallel IS NOT DEFINED THEN
    SET parallel = default_parallel
  END IF

  # Check for recursive multi-mode (depth-based)
  IF depth IS DEFINED AND current_depth IS DEFINED THEN
    GOTO recursive_multi
  END IF

  # Choose dispatch based on parallel flag
  IF parallel = true THEN
    GOTO parallel_dispatch
  END IF

  # Sequential dispatch (default)
  GOTO sequential_dispatch

sequential_dispatch:
  # Process names sequentially using recursive FOLLOW
  SET results = []
  SET success_count = 0
  SET idx = 0

  FOR item IN names
    SET output_file = output_dir + "/greeting_" + idx + ".txt"
    SET result = FOLLOW skill "hello-world" WITH name = item, output_path = output_file
    SET results = APPEND(results, result)
    IF result.status = "success" THEN
      SET success_count = success_count + 1
    END IF
    SET idx = idx + 1
  NEXT item

  # In recursive mode, also delegate to next depth level
  IF depth IS DEFINED AND current_depth < (depth - 1) THEN
    SET next_depth = current_depth + 1
    SET next_names = []
    SET idx2 = 0
    FOR item IN names
      SET next_names = APPEND(next_names, item + "_" + next_depth)
      SET idx2 = idx2 + 1
    NEXT item

    SET sub_output_dir = output_dir + "/depth_" + next_depth
    SET depth_result = FOLLOW skill "hello-world" WITH names = next_names, output_dir = sub_output_dir, parallel = parallel, depth = depth, current_depth = next_depth

    RETURN status = "completed_recursive", results = results, count = success_count, depth_result = depth_result, level = current_depth
  END IF

  RETURN status = "completed", results = results, count = success_count

parallel_dispatch:
  # Process names concurrently using SPAWN
  PARALLEL
    SET idx = 0
    FOR item IN names
      SET output_file = output_dir + "/hello_" + idx + ".txt"
      SET results[idx] = SPAWN skill "hello-world" WITH name = item, output_path = output_file
      SET idx = idx + 1
    NEXT item
  END PARALLEL

  # In recursive mode, delegate to next depth level
  IF depth IS DEFINED AND current_depth < (depth - 1) THEN
    SET next_depth = current_depth + 1
    SET next_names = []
    SET idx2 = 0
    FOR item IN names
      SET next_names = APPEND(next_names, item + "_" + next_depth)
      SET idx2 = idx2 + 1
    NEXT item

    SET sub_output_dir = output_dir + "/depth_" + next_depth
    SET depth_result = FOLLOW skill "hello-world" WITH names = next_names, output_dir = sub_output_dir, parallel = parallel, depth = depth, current_depth = next_depth

    RETURN status = "completed_parallel_recursive", results = results, depth_result = depth_result, level = current_depth
  END IF

  RETURN status = "completed_parallel", results = results

recursive_multi:
  # Recursive multi: builds depth hierarchy
  SET next_depth = current_depth + 1
  SET results = []
  SET success_count = 0
  SET idx = 0

  FOR item IN names
    SET output_file = output_dir + "/d" + current_depth + "_greeting_" + idx + ".txt"
    SET result = FOLLOW skill "hello-world" WITH name = item, output_path = output_file
    SET results = APPEND(results, result)
    IF result.status = "success" THEN
      SET success_count = success_count + 1
    END IF
    SET idx = idx + 1
  NEXT item

  # Delegate to next depth level if we have more depth to go
  IF current_depth < (depth - 1) THEN
    SET next_names = []
    SET idx2 = 0
    FOR item IN names
      SET next_names = APPEND(next_names, item + "_" + next_depth)
      SET idx2 = idx2 + 1
    NEXT item

    SET sub_output_dir = output_dir + "/depth_" + next_depth
    SET depth_result = FOLLOW skill "hello-world" WITH names = next_names, output_dir = sub_output_dir, parallel = parallel, depth = depth, current_depth = next_depth

    RETURN status = "recursive_level", results = results, count = success_count, depth_result = depth_result, level = current_depth, next_level = next_depth
  END IF

  # Base case: no more depth
  RETURN status = "recursive_base", results = results, count = success_count, level = current_depth

tree_dispatch:
  # Build a tree of delegations
  # Each node creates branch_factor children, recursively to depth
  SET current_depth = 0

  # Root node creates greeting
  SET root_file = output_dir + "/tree_root.txt"
  CALL tool "write" WITH path = root_file, content = "Hello, " + tree_root + "!"

  SET children_results = []
  SET idx = 0

  # Create children based on branch_factor
  IF current_depth < depth THEN
    SET child_names = []
    SET child_idx = 0

    WHILE child_idx < branch_factor
      SET child_name = tree_root + "_child_" + child_idx
      SET child_names = APPEND(child_names, child_name)
      SET child_idx = child_idx + 1
    END WHILE

    IF parallel = true THEN
      # Parallel dispatch for children
      PARALLEL
        SET child_results_idx = 0
        FOR child IN child_names
          SET child_output_dir = output_dir + "/node_" + child
          SET current_depth_for_child = current_depth + 1
          SET children_results[child_results_idx] = SPAWN skill "hello-world" WITH tree_root = child, output_dir = child_output_dir, depth = depth, branch_factor = branch_factor, parallel = parallel, current_depth = current_depth_for_child
          SET child_results_idx = child_results_idx + 1
        NEXT child
      END PARALLEL
    ELSE
      # Sequential dispatch for children (deep recursion!)
      SET child_results_idx = 0
      FOR child IN child_names
        SET child_output_dir = output_dir + "/node_" + child
        SET current_depth_for_child = current_depth + 1
        SET child_result = FOLLOW skill "hello-world" WITH tree_root = child, output_dir = child_output_dir, depth = depth, branch_factor = branch_factor, parallel = parallel, current_depth = current_depth_for_child
        SET children_results = APPEND(children_results, child_result)
        SET child_results_idx = child_results_idx + 1
      NEXT child
    END IF
  END IF

  RETURN status = "tree_node", name = tree_root, level = current_depth, children = children_results, child_count = LENGTH(children_results)

END SKILL
```
