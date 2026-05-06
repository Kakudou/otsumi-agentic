# **Atomic Code, Atomic Review**

## *Survive Merge Conflicts. Burn Legacy. Fight Sloppiness.*

## WARNING , HARD REQUIREMENTS
- Never ever push anythings, just local commit !
- Keep commit message on one short sentence !
- Don't add description to a commit, only the summarise line message.

---

## **Atomic Commits: Not Small. Just Focused.**

> "A good commit is mergeable, testable, and understandable, on its own."
> 

- **Atomic means:**
    - One **logical unit of change**
    - Working state (builds, passes tests)
    - Can be reverted/merged independently

- **Why It Matters:**
    - Easier rollback
    - Better PR reviews
    - Cleaner logs
    - Enables  deploys on commit

- **Examples:**
    
    ```
    🔐 feat(auth): add token encryption
    🔧 fix(config): default port from env
    ✅ test(core): add test for calculate_checksum()`
    
    ```
    
    **Anti-pattern:**
    
    ```
    🔐💄 feat(auth + proxy): add token AND config proxy AND fix lint`
    
    ```
    

> One commit = One evolution.
Multiple evolutions? Split them or it becomes a timebomb.
> 

---

## **Save Commits : Legit, But Temporary**

> "You can use checkpoints. You don’t get to dump them on main."
> 

- **Use `🚧 save(scope): message` to:**
    - Frequently back up progress
    - Collaborate in parallel without stress
    - Isolate risky experiments

    ```bash
    # Dev Phase :
    🚧 save(parser): experimenting with whitespace trim
    🚧 save(parser): added fallback for nulls
    # Final Commit: (after squash/rebase)
    🔀 fix(parser): trim whitespace and handle null safely
    
    ```
    

> “You’re allowed to save mess. You’re not allowed to merge mess.”
> 

---

## **Commit Messages: Not Optional, Not Afterthoughts**

> "You’re not writing code for now. You’re writing code for six months from now, when no one remembers."
> 

- **Standard Formats:**
    - `type(scope): message` ← Conventional Commits

- **Commit = contract**:
    - Describes intent
    - Explains scope
    - Can be parsed by changelogs, CI, release scripts

- **Examples:**
    
    ```
    🐛 fix(proxy): prevent crash with null headers
    ✅ test(api): add test for expired tokens
    📌 chore(ci): bump version to 1.2.0
    
    ```
    

- **Bad Examples:**
    
    ```
    fix proxy
    final fix
    maybe this works
    stuff I forgot
    
    ```
    

> Message is the interface between human and history. Treat it with the same rigor as code.
> 

---

## **Why This Isn’t Just Style: It’s Operational Survival**

> "Atomic commits. Iterative reviews. Edge-case tests. Clean names."
> 
> 
> It’s not about looking smart. It’s about making **code evolve without dragging corpses behind it.**
> 
- **This approach:**
    - Makes your code **easier to trust**
    - Makes your changes **easier to review**
    - Makes your failures **easier to revert**
    - Makes your life **easier to maintain**

- **And in a team?**
    - Raises the floor
    - Avoids hero coding and lone-wolf bugs
    - Aligns standards
    - Empowers **collaboration at scale**

---

## **Gitmoji Discipline: One Emoji to Rule the Diff**

> "The emoji is the commit’s type signature. Misuse it, and your history turns to noise."
> 

- **Why Use Gitmoji?**
    - Forces **categorization** of intent adding deeper information
    - Makes logs **instantly scannable** by humans
    - Clean signal in noisy history
        
        
        Gitmoji is a project providing a cheatsheet of emoji with meaning.
        
        Using semantic commit `type(scope): message` is really good, but most of the time, `type` is a macro representation of the content of the commit.
        
        What i mean, if you look for documentation about semantic commit nomenclature you will quickly realized that the `type` is a short list of action like:
        
        ```
        chore:
          For commits that complete routine/automated tasks such as upgrading dependencies.
        
        deprecate:
          For commits that deprecate functionality.
        
        feat:
          For commits that add new functionality to Blockly.
        
        fix:
          For commits that fix bugs/errors in Blockly.
        
        refacto:
          For commits that refactor part of the code.
        
        save:
          For commits that live temporary
        
        ```
        
        For example our code got reviewed by colleague who identified dead code in `connector.py.`
        
        We fix this and following the semantic commit message we do a clean:
        `fix(connector): blabla after review blabla fix blabla`
        
        This is a good commit message, we know a fix occur on connector, but, we can't really rely on message coherence since this is not normalized inputs.
        
        Using gitmoji (let's use the official meaning for now), we can add information:
        
        ```python
        
        ⚰️fix(connector): blabla after review blabla fix blabla
        ```
        
        and if we refer to the cheatsheet meaning of that emoji:
        
        ```
        ⚰️:coffin: Remove dead code
        
        ```
        

- **Identify a list of useful Gitmoji for our usecase**
    
    Using gitmoji has a small learning curve, since we need to remember what emoji mean what.
    That's why i like to create, as a team, a cheatsheet for our needs.
    Like we could do something more connector related:
    
    ```
    🔀:twisted_rightwards_arrows: Code related to convertion to stix
    🧬:dna: Code related to stix2.1 mapping
    🛂:passport_control: Code related to api fetching
    
    ```
    
    This way building a short list of useful emoji could benefit.
    like if i see 🧬 maybe this is related to a bug in the usage of sdk models, while 🛂 is more business related.
    
    so if in a PR for GTI i see:
    
    ```
    🧬refactor(vulnerabilities): blabla bla bla
    🛂fix(vulnerabilities): bla bla bla
    
    ```
    
    Maybe someone who know well the SDK could review the refactor, while someone knowing the API can review the other one.
    

- **Misuse Pattern:**
    
    > 🔧🐛💥 fix all the things ← ambiguous, not automatable, brain poison
    > 

- **Correct Usage:**
    
    ```
    ✨ feat(api): add refresh token logic
    🐛 fix(router): avoid redirect loop in edge case
    ♻️ refactor(core): extract date validation to utils
    
    ```
    

**Coherence Test:**

> If you hesitate between emojis → your commit is doing too much.
> 
> That hesitation is gold. It’s telling you: *break it down.*
> 

## **Fixup Commits: Precision Strikes, Not Junk Shots**

> "Don’t repaint the wall. Patch the crack. Then fold it back like it was always solid."
> 

### **What a Fixup Is**

- Git magic: `git commit --fixup=<hash>` → laser-guided to the right spot.
- A **targeted correction** tied to a specific commit.
- Doesn’t live forever → it’s **temporary scaffolding** for review, squashed before merge.

### **Why Use Fixups**

- **Atomic Reviews**: Reviewers see *exactly* what changed in response to feedback.
- **Traceability**: Fixup is linked to the original commit, no hunting, no “which change was that?”
- **Clean Final History**: Once autosquashed, your log looks like you wrote it perfect the first time.
- **Iterative Safety**: You can stack multiple fixups without rewriting history mid-review.
- **How It Plays Out**
    
    ```bash
    # Reviewer: "Null check missing in mapper" 
    git add mapper.py git commit --fixup=18f9dc5e  # target the mapper commit git push                     
    # reviewer sees one precise fixup
    ```
    
    - More comments? More fixups.
    - Approval hits? 🔥 Fold them all back:
    
    ```bash
    git fetch origin 
    git rebase -i --autosquash origin/main 
    ```
    

### Pros

- Ultra-focused diffs for reviewers
- Auto-linked context to the original commit
- No more “fix after review” noise in history
- Encourages atomic discipline without blocking iteration

### Cons

- Temporary history can look messy until the cleanup
- Requires team to be Git-savvy (basic `rebase -i` know-how)

### **Anti-Pattern**

```bash
🐛 fix: apply review comments 
🐛 fix: apply review comments 2 
🐛 fix: apply review comments FINAL
```

That’s not history, that’s graffiti. 🚫

### **Correct Pattern**

```bash
⚰️ fixup! feat(mapper): add campaign mapping rules 
⚰️ fixup! fix(connector): handle null headers safely
```

### **The Mantra**

History should look inevitable, not improvised.

---

## Burn This In

> “Bad habits don’t break in silence. They break in fire.”
> 

- **Summarize the Propositions:**
    - Big PRs break people, split them
    - Commit for memory, not just merge
    - Test where it fails, not where it runs
    - Gitmoji is your discipline, not your vibe
    - Save work, but clean it before you ship

**Final Mantra:**

> “Code like someone else has to maintain it during a meltdown.
Review like the future depends on it.
Commit like it’s being read by a dumber version of yourself.”
> 

---

## Some Links

[https://gitmoji.dev](https://gitmoji.dev/)
