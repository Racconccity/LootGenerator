# LootGenerator
> **Changes in version 3.2:** Added a [--test](#--test) parameter to the show command. Added default setting for coins (excluding 'no-coins'). Added gaming sets to the Mundane Items, as well as adding a few missing tools. Corrected the Dragon Scale Mail entry to correctly provide a random dragon type, and the armor of resistance entries to correctly provide a random damage type. Removed a few cursed items that had snuck in uninvited.
>
> Virtually all items in the database whose name contained parenthesis was modified to remove them, bringing them in line with the documented recommendations and preparing for a major upgrade currently in the works. If you have customized the database, it is **highly recommended** that you rename your modified handouts to preserve the contents, [reset the database](#initial-setup), export the new database, then re-customize the new handouts using the old ones. (Spells, Gems, and Art have not been changed. If you have customized these, you can simply delete the new ones and rename the old ones appropriately.)

This [Roll20](http://roll20.net/) script generates loot according to the treasure tables in the Dungeon & Dragons 5th Edition Dungeon Master's Guide (DMG). It generates a random number (1d100) and displays the results in chat with options for showing the name of the character discovering the treasure, the name of the object from which the loot is taken, and more. LootGenerator will let you add your own special item to the generated loot, provides options for modifying the generation of each loot category, and allows you to import custom items to all loot categories.

As an improvement to the basic treasure tables, LootGenerator also includes a Mundane Items category in addition to the default Coins, Gems, Art, and Magic Items categories. The default Mundane Items include Adventuring Gear from the Player's Handbook.

LootGenerator is for use with the [5e Shaped Sheet](http://github.com/mlenser/roll20-character-sheets/tree/master/5eShaped) and the D&D 5th Edition OGL Sheet.

## Table of Contents
- [Initial Setup](#initial-setup)
- [Configuration](#configuration)
- [Notable Exclusions](#notable-exclusions)
- [Distributing Loot](#distributing-loot)
- [Script Integration](#script-integration)
- [The Show Command](#the-show-command)
   - [--type](#--type)
   - [--loc](#--loc)
   - [--recip](#--recip)
   - [--incl](#--incl)
   - [--mod](#--mod)
   - [--whisper](#--whisper)
   - [--test](#--test)
   - [examples](#examples)
- [Exporting & Importing](#exporting--importing)
- [Custom Items](#custom-items)
   - [Magic & Mundane Items](#magic--mundane-items)
   - [Built-in Replacement Variables](#built-in-replacement-variables)
   - [Custom Replacement Variables](#custom-replacement-variables)
   - [Spells](#spells)
   - [Gems & Art](#gems--art)
   - [Formatting Guidelines](#formatting-guidelines)

## Initial Setup
Prior to first use, you **must** run this function to populate the loot database with items found in the treasure tables in the DMG and items from the PHB. After installation, you will receive a dialog with a button for running this command.

Unique magic items are automatically removed from the loot database after generation, so a reset option is provided to re-populate the database if necessary. To do so, pass the `--reset` modifier after the `--setup` command.

```
!loot --setup
!loot --setup --reset
```
If you wish to reset the database to start another campaign and you have already [customized](#custom-items) any of the tables, you can simply [re-import](#exporting--importing) data from those tables.

## Configuration
Provides an interface for setting defaults on Coins, Gems, Art, Mundane and Magic Items (see [--mod](#--mod) for more info), for setting whether to show PotionManager and GearManager descriptions to players, and for [exporting tables](#exporting--importing).

```
!loot --config
```

## Notable Exclusions
Because LootGenerator in designed to share item identities with players, all _cursed items_ are absent from the database. If you wish to include cursed items you may do so in many ways. The first (and recommended method) is to simply choose a normal item on the fly from the generated results to be secretly cursed. Most SRD/DMG cursed items are simply cursed versions of another item, so this method allows for the misidentification aspect of the requisite cursed version. The second way is to add the cursed item in with your [imported](#exporting--importing) custom items, and thirdly you can include it as a special item in the [`--show`](#the-show-command) command.

Finally, because of its large size the _Apparatus of Kwalish_ is not in the database. For the same reason, large and/or bulky Mundane Items such as the 10-foot ladder are also not included.

## Distributing Loot
Each call of the show command ([below](#the-show-command)) creates a Treasure Collection that is remembered until all treasure in the Collection has been bestowed or saved. If you do not wish to take the time to distribute treasure when it is found, you can return to it at a later time in various ways.

You may use the `!loot --list` command to display a list of undistributed Treasure Collections, and logging into your game will also automatically generate a list as a reminder. The [config menu](#configuration) also reminds you of undistributed Treasure Collections and provides a link to display the list for players along with the GM's "Bestow" dialog.

Each Treasure Collection is named based on the title generated by the show command. For instance, if you generate loot with the header "Loot from Pirate's Chest", The Treasure Collection will be remembered as "Loot from Pirate's Chest". Multiple uses of the same name will be numbered, so that another pirate's chest would give "Loot from Pirate's Chest 2". It is advisable, of course, to give each one a unique name from the start to avoid confusion later on.

In the "Bestow" dialog displayed immediately after loot is generated, coins and treasure are in separate sections. Coins can be given to a selected character or, if [PurseStrings](#script-integration) is installed, either added to the individual character's Purse or distributed to Party Members. Treasure items can be given individually by clicking the name of the item with a token selected, or given as a whole to the selected character. If PotionManager and/or GearManager are installed, they will add all items that exist in their respective databases including any homebrew potions and custom items you may have added to those scripts. You will also get a link for displaying the description of those items which can be sent to the GM only or to all players (set in the [config dialog](#configuration)).

Items not added via PotionManager or GearManager are written to the character sheet. For the 5e Shaped sheet, it is recorded in the Miscellaneous Notes field near the bottom. For the 5th Edition OGL sheet, it is recorded in the Treasure field in the Bio tab. In either case, the list of loot items will be preceded by name of the Treasure Collection in all caps followed by a colon so that players are reminded of where the items were found.

The final button in the "Bestow" dialog allows all remaining coins and items to be written to a "Party Loot" handout using the same name-and-colon convention above. Be aware, items in the handout can only be manually added (or dragged from the Compendium) to character sheets, so this function is best used as backup or for moving unbestowed treasure to another game rather than a primary loot distribution method.

## Script Integration
LootGenerator will detect the presence of [PurseStrings](https://github.com/blawson69/PurseStrings) and will provide the GM a link for distributing coins. Also, if [PotionManager](https://github.com/blawson69/PotionManager) and/or [GearManager](https://github.com/blawson69/GearManager) are installed, LootGenerator will distribute associated items to characters according to those scripts. If you are using an older version of those scripts, you will be prompted to update them if you want to use their functionality.

## The Show Command
This command is the meat of the LootGenerator script. It generates treasure randomly based on the DMG Treasure tables, plus the Mundane Items, and can be modified by various parameters described below. Each parameter begins with double dashes, and uses a colon to separate the command from its contents where applicable.

The `--show` command must follow `!loot`, but the remaining parameters can be called in any order. See [below](#examples) for examples.

To display help for formatting this command in chat, use `!loot --help`.

#### --type
*Mandatory.* There are 2 general types of loot: *Individual* and *Horde*. There are also 4 treasure levels based on the Challenge Rating (CR) of the monster(s)/NPC(s) from which the loot is coming: level 1 for CR 0-4, level 2 is CR 5-10, level 3 is CR 11-16, and level 4 is CR 17 and higher. The type and level are given together. For instance, passing `--type:Indiv1` will generate Individual loot for CR 0-4. `--type:Horde2` will generate Horde loot for CR 5-10, etc.

#### --loc
*Optional.* This parameter is used to provide the name of the place where the loot was found. By sending `--loc:Pirate's Chest` you make the header read "Loot from Pirate's Chest". By default, the dialog sent to chat will display a simple header of "Loot".

#### --recip
*Optional*. By default, the dialog sent to chat will simply show the list of loot items found. If you wish to indicate the name of the player who discovered the loot, use `--recip:` plus the character's name to display the name of the recipient before the list of loot items.

#### --incl
*Optional.* Often you will want to include a special item in the list of items found. This could be a special quest item, a mysterious note, etc. For example, using `--incl:Cake of Surprise` will add "Cake of Surprise" to the loot found. If you want your special item to have emphasis, you may surround the text with asterisks. Sending `--incl:*Cake of Surprise*` becomes "*Cake of Surprise*" in the output.

If you wish to pass more than one item, you may separate items with a comma. Note that if you wish to use emphasis on these items you will need *each individual item* enclosed in asterisks. For example, use `--incl:Cake of Surprise,Birthday Present` or `--incl:*Cake of Surprise*,*Birthday Present*`.

Note: Special items will always be mixed in with all other items.

#### --mod
*Optional.* This parameter is used to override the [defaults](#configuration) for showing the Coins, Gems, Art, Mundane Item, and Magic Item categories, allowing you to fine tune or customize the loot generated. To eliminate a category, send "no-" and the category of item you wish to skip. Sending `--mod:no-gems` will prevent LootGenerator from generating gems. If your default is not to show Art items, for instance, you can send `--mod:show-art` to make the script generate art objects for the loot.

You may also modify the results of the different categories by using the "less-" and "more-" prefixes. This will subtract or add, respectfully, 25 points from the die roll for each loot category you wish to modify. For instance, sending `--mod:more-coins` will add 25 to the die roll for the results of the Coins. In this example, if the script generates a die roll of 50, it will use 50 for every other loot category and 75 for the Coins. Keep in mind this does not double or halve the actual number of Items or Coins, it only returns results based on a much higher or lower die roll. For Magic Items, using "more-" could result in fewer but more powerful items.

You can modify more than one category by sending multiple parameters separated with a comma. Sending `--mod:no-gems,show-mundane` will override any defaults to Gems and Mundane Items to skip Gems and include Mundane Items.

The possible parameters for this command are:
* 'no-gems', 'less-gems', 'more-gems', and 'show-gems'
* 'no-art', 'less-art', 'more-art', and 'show-art'
* 'no-mundane', 'less-mundane', 'more-mundane', and 'show-mundane'
* 'no-magic', 'less-magic', 'more-magic', and 'show-magic'
* 'no-coins', 'less-coins', 'more-coins', and 'show-coins'

Note: In keeping with the DMG guidelines, Gems, Art, and Magic Items are *only* available as Horde items. Passing `--mod:show-gems` with the `--type:Indiv1` parameter will still not generate Gems. Coins are the bare minimum for any loot, so 'no-coins' is not an accepted default for Coins.

#### --whisper
_Optional._ This parameter allows the GM to skip output of the results to all players. When `--whisper` is given as a parameter, the GM-only ["Bestow Items" dialog](#distributing-loot) is all that is displayed. Everything else functions as normal.

#### --test
_Optional._ This parameter allows the GM to perform a "test run" on a show command. This parameter will only whisper the results to the GM. Generated treasure will not [be saved](#distributing-loot) and unique items will not [be removed](#initial-setup).

#### Examples
```
!loot --show --type:Horde3
!loot --show --type:Indiv1 --mod:less-gems,no-art
!loot --show --loc:Dragon's Lair --type:Horde4
!loot --show --type:Indiv2 --recip:Pip the Pickpocket --incl:Love Letter to Bryon
!loot --show --type:Horde1 --mod:more-mundane --whisper
!loot --show --incl:King's Footlocker --type:Horde3 --mod:more-coins,more-magic --test
```

## Exporting & Importing
To provide customization options for Magic Items, Mundane Items, and Spells, export and import options are provided. You must first export the data into handouts if you wish to add, remove, or modify items in the database. This will give you the proper format for your additional items and provides handouts with the proper titles that the import function will look for.

**Note:** You _must_ run [`--setup`](#initial-setup) before exporting.

```
!loot --export
```

Once you have exported the tables you can edit them to your liking. Leave any original items you wish to use, delete those you don't want, and add your own [custom items](#custom-items).

The required `--tables` parameter for the `--import` command is a comma delimited list of handouts to which you have made changes. The options are *Gems, Art, Table A, Table B, Table C, Table D, Table E, Table F, Table G, Table H, Table I, Mundane*, and *Spells*.

```
!loot --import --tables:Spells
!loot --import --tables:Table A, Mundane
```

If you wish to reset the database to start another campaign and you have already [customized](#custom-items) any of the tables, you can simply re-import data from those tables.

## Custom Items
LootGenerator provides a robust architecture for including homebrewed items of all sorts. Follow the guidelines below and use the default items as examples.

#### Magic & Mundane Items
Magic & Mundane Items have a specific format that allows a weighted distribution. For instance, Items with a weight of 2 will be twice as likely to be encountered as an item with a weight of 1. The default Magic Items have a weight value based on the d100 roll from the DMG tables, so be aware that adding a large number of new items will skew those weights accordingly. Mundane Items are weighted on their general usefulness, size, cost, etc.

There are many Magic Items that are unique and can only be found once during any campaign. To indicate this quality for your custom Items, send "unique" as the last parameter of the Magic Item. This item will then be removed from the database after it has been generated.

The format for Magic & Mundane Items is "weight|name" or "weight|name|unique", each on a separate line in the handout.

#### Built-In Replacement Variables
The Magic & Mundane Items tables use a built-in replacement syntax that allows randomization of item names and the rolling of dice. You will encounter many of these in the [exported handouts](#exporting--importing), but only a few (below) will be relevant for use in your custom Items. Words surrounded by %% are randomized selectors, while any die expression such as 1d4 will be within @ signs.
* **%%damage_types%%** will return a random damage type, such as Acid, Fire, or Necrotic.
* **%%monster_types%%** will return a random monster type, such as Beast, Dragon, or Giant.
* **%%swords%%** will return either Shortsword, Longsword, or Greatsword for magic swords.
* **%%ammo%%** will return Arrow, Crossbow Bolt, Sling Bullet, or Blowgun Needle. Note that these are singular, so that any time you wish to produce multiples you will need an s after.

Examples:

| Database Entry Name| Generated Loot|
| ------------- |-------------|
| *%%swords%%* of Yawning |Longsword of Yawning|
|Potion of *%%damage_types%%* Breath|Potion of Thunder Breath|
|Homing *%%ammo%%*s (*@1d4+1@*)|Homing Crossbow Bolts (3)|

#### Custom Replacement Variables
You can use unique randomization in your own custom items by using $$ around a list of words or phrases separated by a tilde (~). LootGenerator will choose randomly from that list every time the custom item is generated. You may also use any of the built-in replacement variables inside your random options list to give it even more flavor.

Examples:

| Database Entry Name| Generated Loot|
| ------------- |-------------|
|Wand of *$$Yawning\~Winking\~Belching$$* |Wand of Winking|
|*$$@1d6+1@ Large~@1d8+2@ Medium~@1d10+3@ Small$$* Animal Bones|7 Small Animal Bones|
|Feather of *$$%%ammo%%~%%damage_types%%$$* Deflection|Feather of Blowgun Needle Deflection|

*Be careful when editing exported handouts! Leaving out a replacement variable character, or using one in an Item description without actually using it as a replacement variable, can break the script.*

#### Spells
The Spell tables are not weighted as the Magic Items are, but are a simple list of all available spells that are used to generate spell scrolls. Each level of spell is a comma delimited list on one line with a heading designating the level of the spells in the list. These headings are in ALL CAPS followed by a colon and also on their own line. "CANTRIPS:" or "0 LEVEL:" are allowed for cantrips, while the remainder should be "1ST LEVEL:" and so on.

#### Gems & Art
As with the Spell tables, these are simple lists of items based on the DMG tables. There are five levels based on perceived value with the headings in all caps followed by a colon. First level has the heading of "LEVEL 1:" and so on.

#### Formatting Guidelines
- Avoid commas in your custom Item names. They are used to separate items in the generated list of items. You will notice that default Item names that contain commas in the DMG tables have been modified to remove them. For instance, "Shield, +1" has been changed to "+1 Shield".
- Avoid parenthesis. These are used by LootGenerator to denote multiples of items. Three Potions of Healing are generated as "Potion of Healing (3)." If an item has options, use a hyphen instead, i.e. "Ioun Stone - Protection."
- Avoid the use of all replacement variable and syntax characters (:, |, %, @, $, ~).

---
_This script and its contents are permissible under the Wizards of the Coast's [Fan Content Policy](https://company.wizards.com/fancontentpolicy). Portions of the data used are property of and © Wizards of the Coast LLC._
