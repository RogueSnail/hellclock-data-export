# Damage Logic
## What's the damage source?
At the moment there are only 3 places in our game that can create a damage source:
- Skills
- Damage Areas (a lot of Damage Areas will probably use the skill to source a damage instead of the area)
- Status Effects (DoTs and Life Link to be more precise)

It's safe to assume that most of the damage come from Skills

## 1. Defining the *Damage Distribution*
Damage distribution defines 2 things for each damage type (Phys, Fire, Lightning, Plague):
- How much base damage from the attacker we should use for each type
- How much flat damage should we add for each base damage, based on the damage type

Almost all damage distribution is very straightforward, usually looks something like this:
```
Phys Distribution: 100%
Additional Phys: 0

Fire Distribution: 0%
Additional Fire: 0

Lightning Distribution: 0%
Additional Lightning: 0

Plague Distribution: 0%
Additional Plague: 0
```

The distribution is limited to 100% across all types, so it's impossible to have something like
100% Fire Damage and 100% Physical Damage

### 1.1 Skill Base Damage Distribution
All skills have a primary damage type but support modifiers for each *Damage Type Distribution %* and *Additional Damage Type*, those modifiers are used to divert the damage distribution from the primary damage type to another type.

Altough most skills don't receive modifiers to their distribution, there are some cases, like *Blunderbuss* that "override" a skill damage type by forcing a distribution that redirects all damage type from the primary to another type.

Let's take *Blunderbuss* for example

```
Split Shot Primary Damage Type is FIRE, so the distribution looks like this:
{ 'Fire Dist.': 100% }
(we won't show other types since they are all 0)


Blunderbuss adds the following effects to Split Shot
+ 100% to Physical Damage Distribution
So the distribution looks like this:
{ 'Phys Dist.': 100% }
```

Usually the game uses the skill distribution modifiers to override that skill damage type, but there are some cases were it actually creates a mixed damage distribution.

Let's take *Silver Bullets* for example

```
Silver bullets adds the follow effects to all Marksman skills during their Build Damage phase
+ 5% of your Max Conviction to Additional Lightning Damage

Let's assume Split Shot is being fired and the Attacker has 300 Max Conviction.
Split Shot damage distribution should look like this
{ 'Fire Dist.': 100%, 'Additional Light. Damage': 15 (300 * 5%) }
```

### 1.2 Damage Conversion via Stats
If the attacker has any damage conversion stat, for example *Physical Damage Converted as Fire*, the damage distribution will attempt to move all damage distribution and flat damage from the Original Type to the Converted type.

For example
```
Let's assume you have 100% Phys to Fire conversion
If your damage distribution was originally
{ 'Phys Dist.': 100%, 'Additional Phys': 30 }
That would become 
{ 'Fire Dist.': 100%, 'Additional Fire': 30 }

Now let's assume you have 100% Phys to Lightning and 100% Lightning to Fire
If you damage distribution was originally
{ 'Phys Dist.': 100% }
That would also become 
{ 'Fire Dist.': 100% }

If you damage distribution was originally
{ 'Plague Dist.': 100%, 'Additional Lightning': 30 }
That would become 
{ 'Plague Dist.': 100%, 'Additional Fire': 30 }

```

Damage conversion follow a specific order to avoid distribution *ping pong*.
The system will always try to convert in the following order (the design will always try respect this order): Physical -> Lightning -> Plague -> Fire


## 2. Trigger *On Before Take Damage* Events for Target
This event is specific to some effects that want to *modify* how you take damage, usually we use this event to mitigate damage.

Examples:
- Take -10%[x] damage from Bleeding enemies
- Take -5%[x] damage while Moving

## 3. Check if it's a critical hit
Only *Direct Hits* (anything that's not Damage Over Time) can cause critical damage.
We check for (Attacker Crit. Chance + Target Additional Chance to Receive Critical) and compare to a random value between 0 and 1

## 4. Calculate the Distributed Damage

### 4.1 Calculate each Damage Base
If the damage type that has no distribution or added damage > 0 it's ignored

Each valid damage type will have the base using this formula:

```
Additional Damage = Additional Damage (from Distribution) * Damage Type Modifiers (Damage %, Damage Type %)

Bonus Damage = Bonus Damage (from Stats) * Damage Type Modifiers (Damage %, Damage Type %) * Skill Damage Mod (each skill has a % damage used)

Base Damage = Base Damage (from Stats) * Damage Distribution (from Distribution) * Skill Damage Mod (each skill has a % damage used) * Damage Type Modifiers (Damage %, Damage Type %)


Damage Value = Additional Damage + Bonus Damage + Base Damage

If the Damage is Critical, the Damage Value will be multiplied by the Critical Damage modifier

```

### 4.2 Apply Damage Type Shift Logic
Damage Shift is used for effects that say "Take X% Phys Damage as Fire" for example

It's pretty straightforward, a % of the damage base value is moved from a base type to another base type


### 4.3 Apply Damage Types to Target

For each damage base with a value > 0 the system will calculate how will the damage base be applied to that target.

The operations follow this order:
1. Check for Damage Immunity, if it's immune, flag damage as ignored
2. Check if it's a DoT, then apply the DoT resistance stat
3. If it's not a DoT, try to apply damage avoidance
    1. If it's not area damage, try to evade
    2. If did not evade, try to use Endurance / Anti Magic to avoid
    3. If did not avoid, try to use the Chance to Miss
4. Apply the Damage Type Resistance to the damage
5. Apply the Damage Reduction to the damage

## 5. Trigger *On Before Deal Damage* Events for Attacker

## 6. Try to Apply Life/Mana Leech

## 7. Apply the Damage to the Target

## 8. Trigger *On Deal Damage* Events

## 9. Try to Apply Ailments

## 10. Check if Target was Killed
