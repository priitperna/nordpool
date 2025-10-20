# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Home Assistant blueprint for automating actions based on Nordpool electricity prices. The blueprint triggers automations when electricity prices cross user-defined thresholds, enabling smart home devices to run during cheap hours and avoid expensive hours.

## Architecture

### Single File Blueprint Structure

The entire automation logic is contained in `nordpool.yaml`, which defines:

- **Blueprint metadata**: Name, description, and domain (automation)
- **Input parameters**: User-configurable settings for the automation
  - `grid_area`: Selector for Nordpool integration sensor entity
  - `expensive_hours`: Number of most expensive hours to avoid (0-24)
  - `cheap_price`: Absolute price threshold for "cheap" electricity
  - `expensive`: Action sequence to run during expensive periods
  - `cheap`: Action sequence to run during cheap periods
- **Trigger**: State changes of the selected grid area sensor
- **Action logic**: Template-based conditions that evaluate:
  - Whether current price ranks in the top N expensive hours (using sorted price list)
  - Whether current price is below the absolute cheap threshold
  - Routes to appropriate action sequence (cheap or expensive)

### Logic Flow

The blueprint uses Home Assistant's templating to compare:
1. Current price against sorted daily price ranking: `state_attr(grid_area_var,'today')|sort`
2. Current price against absolute threshold: `state_attr(grid_area_var,'current_price')|float < cheap_price_var`

The conditions use OR logic - electricity is considered "cheap" if EITHER:
- Current hour is NOT in the top N expensive hours, OR
- Current price is below the absolute cheap threshold

## Working with This Code

### Testing Changes

Since this is a Home Assistant blueprint:
- Test by importing into Home Assistant (Community → Blueprints → Import)
- Validate YAML syntax: `python3 -c "import yaml; yaml.safe_load(open('nordpool.yaml'))"`
- Test requires a Home Assistant instance with Nordpool integration installed

### YAML Structure Notes

- Uses Home Assistant-specific YAML tags: `!input` for blueprint inputs
- Template syntax uses Jinja2: `{{ }}` for expressions, filters with `|`
- State access: `states(entity_id)` for current state, `state_attr(entity_id, 'attribute')` for attributes
- The Nordpool sensor provides: `current_price`, `today` (list of hourly prices), and other time-series data

### Common Modifications

When modifying the logic:
- Price comparisons use `|float` filters to ensure numeric operations
- The sorting logic `(state_attr(...,'today')|sort)[24 - expensive_hours_var |int]` finds the threshold price
- Array indexing is 0-based; index `[24 - N]` gives the Nth highest price in a 24-hour sorted array
- Both conditions should be updated together to maintain logical consistency
