# Engineering Take-Home Challenge - 12/2020
Congratulations on making it to the final round of interviews for our engineering position at
American Efficient!

In order for us to get a better sense of your programming ability,
and to give us a structured task to discuss during the "in-person" interview on January 8th, we
would like you to complete this short take-home challenge. We expect it to take between 4 and 6 hours
to complete. You may spend more time than that, but please don't go too much over that limit as this
is intended to be a measure of your programming ability, not how much free time you have. :smile:
Partial solutions are acceptable.

## Task Description

For this task, we would like you to build a command line tool that can query products in the EnergyStar
database. EnergyStar provides data sets [and an API](https://www.energystar.gov/productfinder/advanced)
that allows developers to create new applications powered by EnergyStar data. This tool should
offer the following commands (for all of these commands, see below for which product categories and
features should be supported):

- `get`: Given a product category, brand name, and model name, fetch the product info from the EnergyStar
database and display its features to the user.
- `search`: Given a product category and the values of one or more features, output a CSV of all
products that satisfy the search criteria.

## Supported Product Categories and Features

- **Televisions**
  - `pd_id`
  - `brand_name`
  - `model_name`
  - `technology_type`
  - `ethernet_supported`
  - `size_inches`
  - `resolution_pixels`
  - `power_consumption_in_on_mode_watts`
  - `maximum_on_mode_power_for_certification_watts`
  - `power_consumption_in_standby_mode_watts`
  - `maximum_standby_passive_mode_power_for_certification_watts`
- **Uninterruptible Power Supplies**
  - `pd_id`
  - `brand_name`
  - `model_name`
  - `active_output_power_rating_minimum_configuration_w`
  - `height_mm`
  - `width_mm`
  - `depth_mm`
  - `topology_ac`
  - `total_input_power_in_w_at_0_load_min_config_lowest_dependency_ac`
  - `efficiency_at_25_load_min_config_lowest_dependency_ac`
  - `efficiency_at_50_load_min_config_lowest_dependency_ac`
  - `efficiency_at_75_load_min_config_lowest_dependency_ac`
  - `efficiency_at_100_load_min_config_lowest_dependency_ac`
- **Electric Vehicle Supply Equipment** (also called Electric Vehicle Chargers)
  - `pd_id`
  - `brand_name`
  - `model_name`
  - `max_nameplate_output_current_a`
  - `input_voltage_v`
  - `no_vehicle_mode_input_power_w`
  - `no_vehicle_mode_total_allowance_w`
  - `no_vehicle_mode_power_factor`
  - `partial_on_mode_input_power_w`
  - `partial_on_mode_total_allowance_w`
  - `partial_on_mode_power_factor`
  - `idle_mode_input_power_w`
  - `idle_mode_total_allowance_w`
  - `idle_mode_power_factor`
  - `full_current_operation_mode_test_total_loss_w`

In addition to the features listed above, which are provided by EnergyStar, we would like you to
calculate a `delta_watts` feature for each product category. These delta watts represent the amount of
energy we can trade into the market for each product. The equations for how to calculate delta watts
are listed below.

## Delta Watts Equations

_Please note: These are not the actual equations. We have completely made them up for this task._
### Televisions
The delta watts for TVs is the maximum watts allowed by certification (for passive and on modes)
times a scaling factor based on the size of the screen.
```python
delta_watts = scaling_factor *
(
    (maximum_standby_passive_mode_power_for_certification_watts - power_consumption_in_standby_mode_watts)
    + (maximum_on_mode_power_for_certification_watts - power_consumption_in_on_mode_watts)
)
```
where the `scaling_factor` is based on the `size_inches` feature as shown here

| Size  | Factor |
|-------|--------|
| < 20 in |    0.9 |
| < 40 in |   0.85 |
| < 60 in |    0.7 |
| >= 60 in |   0.65 |

### Uninterruptible Power Supplies
The delta watts for an uninterruptible power supply (UPS) is its total input power at 0 load,
times an average of its efficiency at 0%, 25%, 50%, 75%, and 100% load, weighted by how much time
typical UPS devices spend at each load.
```python
delta_watts = total_input_power_in_w_at_0_load_min_config_lowest_dependency_ac *
(
    (pct_time_at_0_load * 1.0)
    + (pct_time_at_25_load * efficiency_at_25_load_min_config_lowest_dependency_ac)
    + (pct_time_at_50_load * efficiency_at_50_load_min_config_lowest_dependency_ac)
    + (pct_time_at_75_load * efficiency_at_75_load_min_config_lowest_dependency_ac)
    + (pct_time_at_100_load * efficiency_at_100_load_min_config_lowest_dependency_ac)
)
```
The `pct_time_at_X_load` values are shown in the following table

| Load | % Time Spent |
|------|--------------|
|    0 |          0.8 |
|   25 |          0.1 |
|   50 |         0.02 |
|   75 |         0.03 |
|  100 |         0.05 |

### Electric Vehicle Supply Equipment
The delta watts for Electric Vehicle Chargers is the total loss in watts during the EnergyStar test
plus the difference between watts used and watts required for certification, scaled by a factor, for
each of: no vehicle, partial on, and idle modes.
```python
delta_watts = full_current_operation_mode_test_total_loss_w
+ ((no_vehicle_mode_total_allowance_w - no_vehicle_mode_input_power_w) * no_vehicle_mode_power_factor)
+ ((partial_on_mode_total_allowance_w - partial_on_mode_input_power_w) * partial_on_mode_power_factor)
+ ((idle_mode_total_allowance_w - idle_mode_input_power_w) * idle_mode_power_factor)
```

# FAQ
## What language should I use?
We would prefer this be written in Python, but you are welcome to use whatever language you are most
comfortable with. Regardless of the language chosen, please provide documentation about how to install
dependencies and run the code.

## What external dependencies can I use?
You are welcome to use open source packages to access the API. The EnergyStar API documentation
recommends [sodapy](https://github.com/xmunoz/sodapy), although you could also use something like
[requests](https://requests.readthedocs.io/en/master/) if you prefer.

Other than that, however, we would prefer that your solution use only things available in the standard
Python library. We recognize that some of this could be done efficiently with `pandas`, but we want
to see a more object oriented approach.

## How do I submit my results?
You can either create a private repository on Github with your solution and email a link or send
a zip of the solution to
[kristen@modern.energy](mailto:kristen@modern.energy) and
[shayne@americanefficient.com](mailto:shayne@americanefficient.com)

## OMG, those field names!
Yeah, EnergyStar gets kind of carried away with how it names the fields in its datasets. We leave it
up to you to create variable names that make sense but are able to be associated with the EnergyStar
data. You are also welcome to choose the appropriate data types for each field - you don't have to stick
with what EnergyStar provides.

## What should I do about non-unique products in the `get` command?
Sometimes `brand_name` and `model_name` will not be enough to fully specify a unique product in the
`get` command. In those cases, please show the first product from the API response and tell the user that
there are more products available (which they should be able to access with the `search` command).

## What should I do about missing data?
Please use your best judgment about how to report results with missing data.

## What should I do if I have any other questions?
Please email
[kristen@modern.energy](mailto:kristen@modern.energy) and
[shayne@americanefficient.com](mailto:shayne@americanefficient.com). We will do our best to answer
your question and also share the question and answer (anonymously) with the other applicants.
