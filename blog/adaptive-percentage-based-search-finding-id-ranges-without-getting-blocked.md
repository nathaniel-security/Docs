---
cover: >-
  https://images.unsplash.com/photo-1606701282529-35614f196fb6?crop=entropy&cs=srgb&fm=jpg&ixid=M3wxOTcwMjR8MHwxfHNlYXJjaHwxfHxhbGdvfGVufDB8fHx8MTc2NjgyMjc0OXww&ixlib=rb-4.1.0&q=85
coverY: 0
---

# Adaptive Percentage-Based Search: Finding ID Ranges Without Getting Blocked

### The Problem

During a test, I discovered that user ID `16000` exists on a web application. But I needed to answer two critical questions:

1. **What's the lower bound?** Where do valid IDs start? 1? 5000? 12000?
2. **What's the upper bound?** Where do they end? 20000? 50000? 100000?

I had one valid ID and needed to find the complete range with:

* **Minimal total requests** (more stealth)
* **Even fewer 404 errors** (WAFs detect and block repeated failures and risk of getting noticed)

**The Challenge:** Modern sites use Web Application Firewalls (WAFs) that block IPs showing automated scanning behavior. Too many 404 errors in a row = instant block.

### The Solution

This algorithm finds both the lowest and highest valid IDs from a single known ID, while staying under WAF detection thresholds.

**Core Concept:** Adjust your search step size based on what you find:

* Find a valid ID (200)? Take **bigger steps** (go faster)
* Hit an invalid ID (404)? Take **smaller steps** and **backtrack** to safety

Think of it like exploring a dark hallway:

1. You know one door opens (your starting ID)
2. Step back 10% of your position
3. Try that door:
   * Opens? Great! Try a bigger step (20%) next time
   * Locked? Go back to the last open door, try a smaller step (5%)
4. Repeat until you find the edges

**Key Formula:** `step_size = percentage × current_id`

### Why Traditional Algorithms Fail

**Linear Search** (`16000, 15999, 15998...`)

* Hundreds of thousands of requests
* Guaranteed block

**Binary Search** (split range in half each time)

* Very fast in theory
* Can generate 10+ consecutive 404s
* **42% block rate** in testing

**Exponential Search** (double distance: `+1, +2, +4, +8...`)

* Overshoots boundaries
* **12% block rate** in testing

None of these were designed with WAF detection in mind.

### How It Works

#### Step-by-Step Rules

```
Starting percentage: 10%
On success (200): Double percentage (max 50%)
On failure (404): Halve percentage (min 0.05%) + wait before next try

Formula: step = percentage × current_id
```

#### Example: Finding Bounds from ID 16000

**Lower Bound (searching backwards):**

```
Start: 16000, step = 10%
→ 14400 ✓ (double to 20%)
→ 11520 ✓ (double to 40%)
→ 6912  ✗ (backtrack, halve to 20%)
→ 9216  ✗ (backtrack, halve to 10%)
→ 10368 ✗ (backtrack, halve to 5%)
→ 10944 ✓ Lower bound found!
```

**Upper Bound (searching forwards):**

```
Start: 16000, step = 10%
→ 17600 ✓ (double to 20%)
→ 21120 ✓ (double to 40%)
→ 29568 ✗ (backtrack, halve to 20%)
→ 25344 ✗ (backtrack, halve to 10%)
→ 23232 ✓ Upper bound found!
```

**Result:** Found both boundaries in \~12 requests with minimal 404s

### Testing Results

We tested against a simulated WAF that blocks after **7 consecutive 404s**.

**Test Setup:**

* 200,000 ID range
* 5 different data densities (20%-99% valid IDs)
* 25 runs per scenario = 125 total trials

**Ban Rates:**

* **Adaptive Percentage:** 0% (0 out of 125 trials blocked)
* **Binary Search:** 42% (53 trials blocked)
* **Exponential Search:** 12% (15 trials blocked)

### Performance

For a 200,000 ID range:

* **Linear Search:** \~200,000 requests
* **Binary Search:** \~17 requests (but high block risk)
* **Adaptive Percentage:** \~8-12 requests (zero blocks)

**95% fewer requests than linear search, with built-in stealth.**

### How It Stays Undetected

1. **Success Bias** - Actively seeks valid IDs, avoids 404 clusters
2. **Backtracking** - Never keeps probing into invalid regions
3. **Variable Steps** - Unpredictable pattern breaks detection heuristics
4. **Exponential Delays** - Waits 1s, 2s, 4s, 8s after consecutive failures

### Implementation Details

**Backtracking:**

* Saves last valid ID position
* Returns there after any 404
* Prevents getting lost in invalid ranges

**Fine-Tuning:**

* At minimum step size (0.05%), switches to precise sequential search
* Confirms exact boundary within ±0.1% of candidate

**Timing:**

* Delays increase with consecutive 404s
* Resets on success
* Mimics human browsing

### When to Use This

**Perfect for:**

* Single IP testing (no proxy pool available)
* Production systems with active WAFs
* Unknown ID ranges
* Time-limited engagements

**Real Constraints It Handles:**

* No IP rotation
* Live WAF monitoring
* Limited time

### The Bottom Line

This algorithm trades a tiny bit of theoretical speed for **massive practical stealth**.

* **Efficiency:** Near O(log n) with small boundary refinement
* **Safety:** 0% observed block rate across 125 trials

When WAFs are in play and you can't rotate IPs, this is the right tool. Standard textbook algorithms will get you blocked.
