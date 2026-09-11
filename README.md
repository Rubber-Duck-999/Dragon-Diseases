# Dragon Diseases (CK3 / AGOT submod scaffold)

Framework for random outbreaks that can infect both humans and dragons, hit dragons much
harder, destroy dragon eggs personally held by an infected character, and stop seeding
once the living dragon population drops below 5. Requires no DLC - CK3 base game + AGOT
(free). Built on CK3's real, built-in epidemics system, not custom infection-rolling logic.

**Status: design scaffold, not tested in-game.** Comments in each file mark what's
confirmed against real AGOT/vanilla/Valyrian Steel script vs. what's still a best-effort
guess.

## How it actually works

Earlier drafts of this mod hand-rolled global variables and on_actions to simulate
infection spread and death rolls. That guesswork got replaced once we found:

- `common/epidemics/00_epidemics.txt` - CK3's real epidemic type registry (confirmed via
  the `bubonic_plague` entry). An epidemic type declares a `trait`, and the engine grants/
  removes that trait on its own as a character enters/leaves an active outbreak in their
  province and passes `can_infect_character` - no manual `add_trait` needed.
- `contract_disease_effect` / `create_epidemic_outbreak` / `can_contract_disease_trigger`
  as real, confirmed effects/triggers, seen in live use in the "Valyrian Steel" submod's
  `fund_expedition.txt` (an explorer returning from Old Valyria can catch smallpox/bubonic
  plague/measles/ergotism this way) and AGOT's own `learning_medicine_events.txt`.

So this mod now works like:

1. **`common/epidemics/00_dragon_disease_epidemics.txt`** registers two epidemic types -
   `wyrmrot` (human strain) and `wyrmrot_dragon` (dragon strain) - each with a `trait`,
   `can_infect_character`, and `on_start`/`on_end` hooks.
2. `can_infect_character` for the human strain reuses vanilla's own
   `can_contract_disease_trigger` (works fine for humans). The dragon strain needs its own
   trigger (`can_contract_dragon_disease_trigger` in
   `common/scripted_triggers/dragon_disease_triggers.txt`), since the vanilla one
   hard-requires `is_human = yes` - the exact thing that keeps dragons disease-free today.
3. Once a character is infected, the epidemic engine handles the trait itself. What it
   *doesn't* handle is death/recovery for a custom trait outside vanilla's hardcoded
   disease-pulse lists - so `on_character_infected` kicks off a small self-rescheduling
   character event (`events/dragon_diseases_events.txt`, ids `dragon_diseases.0001`/`.0002`)
   that rolls death on a delay and keeps re-triggering itself while the character's still
   sick. Same idiom vanilla itself uses for its own disease pulses, just done in script
   since our traits aren't in vanilla's list.
4. That same human-strain event (`dragon_diseases.0001`) also rolls egg destruction each
   pulse, scoped via `every_owned_artifact` to dragon eggs the *infected character
   personally holds* - not every egg in the world. No world-scope iterator needed for this.
5. Outbreak-seeding frequency remains custom on_action logic
   (`common/on_action/wyrmrot_on_actions.txt`) - the epidemic system itself has no concept
   of "start a new outbreak automatically," something has to call `create_epidemic_outbreak`.
6. The population end-condition (`dragon_population_critical_trigger`, `living_dragons`
   count < 5) gates *new* outbreak seeding - no effect was found anywhere to forcibly end
   an epidemic already running, so existing outbreaks just taper off via their own
   `infection_duration`/spread decay once seeding stops.

## Layout

- `common/traits/00_dragon_disease_traits.txt` - the trait pair per disease.
- `common/epidemics/00_dragon_disease_epidemics.txt` - the real epidemic type
  registrations. This is the heart of the mod now.
- `common/scripted_triggers/dragon_disease_triggers.txt` - the dragon-side
  `can_infect_character` check, the egg-exposure check, and the population end condition.
- `common/scripted_effects/dragon_disease_effects.txt` - outbreak-active flag helpers and
  the death-roll / egg-destruction rolls.
- `common/on_action/wyrmrot_on_actions.txt` - what's left outside the epidemic system:
  outbreak seeding frequency only. **Still the least-verified file** - see its header
  comment.
- `events/dragon_diseases_events.txt` - the self-rescheduling death/recovery pulses
  (the human one also rolls egg destruction against that character's own held eggs), plus
  flavour events for outbreak start/end.
- `common/death_reasons/00_dragon_disease_death_reasons.txt` - one entry per disease.

## Adding a second disease

1. Add a new trait pair in `00_dragon_disease_traits.txt`.
2. Add a new epidemic type pair in `00_dragon_disease_epidemics.txt`, pointing
   `on_character_infected` at two new event ids.
3. Add those two self-rescheduling events in `dragon_diseases_events.txt`.
4. Add a `death_<name>` entry in the death_reasons file.
5. No changes needed to the shared trigger/effect files.

## Known open items (see inline comments for detail)

- Whether outbreak-seeding should live in a bare on_action (current draft) or a
  self-rescheduling event like the death pulses - `create_epidemic_outbreak` is only ever
  seen called from inside an event in the reference material we have.
- Confirming `has_variant = dragon_egg` actually matches the artifact type from
  `00_agot_dragonegg_type.txt` (only its slot definition has been checked so far).
- `outbreak_intensities`/`infection_levels` values are placeholders modelled loosely on
  AGOT's `bubonic_plague` entry - needs real balance tuning, not a correctness risk.

## Installing

Move the `dragon_diseases` folder and the `dragon_diseases.mod` file into your CK3
`mod/` directory (`Documents/Paradox Interactive/Crusader Kings III/mod/` by default),
then edit `dragon_diseases.mod`'s `path=` line if you place the folder anywhere other
than `mod/dragon_diseases`. Enable it below AGOT in the launcher's mod list.
