# 🔄 Terraform Lifecycle Rules Lab

In this lab, I learned how to **control Terraform resource creation and destruction** using lifecycle rules, how `keepers` trigger recreation for random resources, and how to understand the implications of `create_before_destroy` and `prevent_destroy`. I also practiced reading state, inspecting resource IDs, and analyzing when new values (like random strings or pets) are generated.

---

## 📋 Lab Overview

**Goal:**

* Understand Terraform lifecycle rules
* Control the order resources are recreated
* Learn how `keepers` affect random provider resources
* Observe how lifecycle affects local file creation
* Use `terraform state` and `terraform show` to inspect resources

**Learning Outcomes:**

* Use `create_before_destroy` safely and correctly
* Understand why some resources can and cannot support this pattern
* Recognize how variable changes influence random resources
* Prevent destruction of important resources using lifecycle rules
* Diagnose resource recreation behaviour using `plan` and `apply`

---

## 🛠 Step-by-Step Journey

### Step 1: Inspect Existing Resource Types

**Directory:**

```
/root/terraform-projects/project-mysterio
```

Resources inside `main.tf`:

* `local_file`
* `random_string`

These are the two resource types used.

---

### Step 2: Create the Resources

Commands executed:

```bash
terraform init
terraform plan
terraform apply   # entered "yes"
```

Both resources were created successfully.

---

### Step 3: Which Resource Was Created First?

**Answer:**
`random_string` was created first.

Terraform handles resources in dependency order—in this case, both were independent, so order is based on internal graph evaluation.

---

### Step 4: Run Plan After Modifying Configuration

After modification:

```bash
terraform plan
```

**Result:**
➡️ **Both resources will be replaced**

---

### Step 5: Why Was the Random String Recreated?

**Reason:**
The `keepers` argument changed.

`keepers` is used by random provider resources to trigger recreation.

If *any* value inside the `keepers` map changes → Terraform forces a new random value.

In this lab, `length` inside `keepers` changed from **10 → 12**.

---

### Step 6: Understanding `keepers`

* Used only by **random provider** resources
* Accepts a **map**
* Any change to any key forces recreation
* Perfect for forcing regeneration based on meaningful controlled variables

---

### Step 7: Change String Recreation Order via Lifecycle

Task: Ensure new random string is created **before** old one is destroyed.

Solution added inside `random_string`:

```hcl
lifecycle {
  create_before_destroy = true
}
```

Applied via:

```bash
terraform apply
# yes
```

This allows seamless recreation.

---

### Step 8: Apply Lifecycle to File Resource Too

Task: Add the same lifecycle block to `local_file`.

Inside `local_file`:

```hcl
lifecycle {
  create_before_destroy = true
}
```

Executed:

```bash
terraform apply
# yes
```

---

### Step 9: Retrieve File Resource ID

To identify the newly created file resource ID:

```bash
terraform state show local_file.file
```

**ID example:**
`e6b65...`

---

### Step 10: Read the File Contents

Command:

```bash
cat /root/random_text
```

**Result:**
`No such file or directory`

Explanation:

Because the filename stayed the same, when `create_before_destroy` was enabled for `local_file`, Terraform tried to:

1. Create new file → conflict
2. So it deleted the old one immediately
3. Then attempted to recreate it
4. Failed to recreate due to simultaneous operations

This shows why **local_file + create_before_destroy** is dangerous unless the filename is unique.

---

### Step 11: Recreate File

Running `terraform apply` again successfully recreated the file, since it no longer existed.

---

### Step 12: Understand When `random_pet` Will Recreate

The configuration now contains only:

```hcl
resource "random_pet" "super_pet" { ... }
```

Question: When will a new pet ID be created?

**Answer:**
When **either** the `length` **or** `prefix` variables change.

Any change to these input variables forces recreation.

---

### Step 13: Prevent the Resource from Ever Being Destroyed

To ensure `super_pet` **cannot be destroyed**, the following lifecycle rule was added:

```hcl
lifecycle {
  prevent_destroy = true
}
```

Applied using:

```bash
terraform apply
# yes
```

Terraform will now *refuse* to destroy this resource—even if `destroy` is called.

---

## ✅ Key Commands Summary

| Task                              | Command                                    |
| --------------------------------- | ------------------------------------------ |
| Initialize project                | `terraform init`                           |
| Preview execution                 | `terraform plan`                           |
| Apply changes                     | `terraform apply`                          |
| Inspect state                     | `terraform show`                           |
| Show a single resource from state | `terraform state show <resource>`          |
| Read file contents                | `cat /root/random_text`                    |
| Add lifecycle rules               | `create_before_destroy`, `prevent_destroy` |
| Recreate random resources         | Modify `keepers` or input variables        |

---

## 💡 Notes / Tips

* `create_before_destroy` is **safe** for cloud resources but **not safe** for local files unless filenames change.
* `keepers` is a powerful mechanism for *controlled* regeneration of random values.
* `prevent_destroy` is extremely useful for protecting:

  * database instances
  * production buckets
  * critical resources
* Random provider resources live only in **Terraform state**, so conflicts are minimal.
* Always inspect state when debugging resource recreation.

---

## 📌 Lab Summary

| Step                        | Status | Key Takeaways                            |
| --------------------------- | ------ | ---------------------------------------- |
| Inspect resource types      | ✅      | Detected `local_file` & `random_string`  |
| Create resources            | ✅      | Both applied                             |
| Observe recreation behavior | ✅      | Keepers change triggered recreation      |
| Add lifecycle rules         | ✅      | `create_before_destroy` applied          |
| Analyze local file conflict | ✅      | Same filename caused deletion            |
| Test random_pet recreation  | ✅      | Changes to `length` or `prefix` recreate |
| Protect resource            | ✅      | Added `prevent_destroy = true`           |

---

## ✅ References

* [Terraform Lifecycle Rules](https://developer.hashicorp.com/terraform/language/meta-arguments/lifecycle)
* [Random Provider Documentation](https://registry.terraform.io/providers/hashicorp/random/latest/docs)
* [Local File Provider Documentation](https://registry.terraform.io/providers/hashicorp/local/latest/docs/resources/file)
* [Terraform State Commands](https://developer.hashicorp.com/terraform/cli/commands/state)
