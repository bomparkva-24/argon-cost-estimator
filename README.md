
```markdown
# Argon Cost Estimator

Estimate the cost of a Gemini 4 Argon workload using fresh input, cached input,
output tokens, and web-search calls.

This is an unofficial calculator. It does not make API calls, and its results are
estimates rather than billing guarantees.

## Rates

Rates observed on CometAPI on November 11, 2026:

| Item | CometAPI | Google list-price comparison |
|---|---:|---:|
| Fresh input / 1M tokens | $1.60 | $2.00 |
| Output / 1M tokens | $8.00 | $10.00 |
| Cached input / 1M tokens | $0.08 | Not compared |
| Web searches / 1,000 calls | $8.00 | Not compared |

Update `RATES` when prices change. The official-price comparison is token-only
and does not include search or other possible charges.

## Usage

```bash
python estimate.py \
  --input 200000 \
  --output 2000 \
  --requests 20 \
  --cached-share 0.95
```

Example:

```text
First request:       $0.3360
Each repeat request: $0.0472
20 requests:         $1.23
No-cache estimate:   $6.72
Official token-only: $8.40
```

`--cached-share` applies only to requests after the first. Cache eligibility
depends on the provider's rules and an unchanged reusable prompt prefix.
```

Replace the script with:

```python
import argparse

MILLION = 1_000_000

RATES = {
    "input": 1.60,
    "output": 8.00,
    "cached": 0.08,
    "search_per_1k": 8.00,
}

OFFICIAL = {"input": 2.00, "output": 10.00}


def nonnegative_int(value):
    value = int(value)
    if value < 0:
        raise argparse.ArgumentTypeError("must be zero or greater")
    return value


def positive_int(value):
    value = int(value)
    if value < 1:
        raise argparse.ArgumentTypeError("must be at least 1")
    return value


def share(value):
    value = float(value)
    if not 0 <= value <= 1:
        raise argparse.ArgumentTypeError("must be between 0 and 1")
    return value


parser = argparse.ArgumentParser(description="Estimate Gemini 4 Argon costs")
parser.add_argument("--input", type=nonnegative_int, required=True)
parser.add_argument("--output", type=nonnegative_int, required=True)
parser.add_argument("--requests", type=positive_int, default=1)
parser.add_argument("--cached-share", type=share, default=0.0)
parser.add_argument(
    "--searches",
    type=nonnegative_int,
    default=0,
    help="web-search calls per request",
)
args = parser.parse_args()

search_cost = args.searches * RATES["search_per_1k"] / 1_000
fresh_input_cost = args.input / MILLION * RATES["input"]
output_cost = args.output / MILLION * RATES["output"]

first_cost = fresh_input_cost + output_cost + search_cost

cached_tokens = args.input * args.cached_share
repeat_fresh_tokens = args.input - cached_tokens
repeat_cost = (
    repeat_fresh_tokens / MILLION * RATES["input"]
    + cached_tokens / MILLION * RATES["cached"]
    + output_cost
    + search_cost
)

total = first_cost + repeat_cost * (args.requests - 1)
no_cache = first_cost * args.requests
official_token_only = (
    args.input / MILLION * OFFICIAL["input"]
    + args.output / MILLION * OFFICIAL["output"]
) * args.requests

print(f"First request:       ${first_cost:.4f}")
print(f"Each repeat request: ${repeat_cost:.4f}")
print(f"{args.requests} requests:         ${total:.2f}")
print(f"No-cache estimate:   ${no_cache:.2f}")
print(f"Official token-only: ${official_token_only:.2f}")
```

Use the requested link exactly:

[Gemini 4 Argon model page and pricing](https://www.cometapi.com/models/google/gemini-4-argon/?utm_source=github&utm_medium=distribution&utm_campaign=bom&utm_content=gemini-4-argon)
