---
title: Detecting Cheaters in Counter-Strike&#58; Global Offensive
description: This tutorial will demonstrate usage of the Nexosis API to build a classification model that predicts if a user should be banned or not. 
copyright: 2017 Nexosis 
layout: default
category: Sports & Games
tags: [Classification, PowerShell, Video Games]
use_codestyles: true
---

[Counter-Strike: Global Offensive](https://en.wikipedia.org/wiki/Counter-Strike:_Global_Offensive){:target="_blank"}, or CS:GO for short, is a team-based [multiplayer](https://en.wikipedia.org/wiki/Multiplayer_video_game){:target="_blank"} highly competitive [first person shooter](https://en.wikipedia.org/wiki/First-person_shooter){:target="_blank"} video game made and published by [Valve Software](http://www.valvesoftware.com){:target="_blank"} in 2012 and is still very popular today. This game, like many online games, has had it's share of issues with cheaters. Since cheaters use different techniques to give them an unfair advantage, we hypothesize a player's in-game performance metrics could be used to build a machine learning model that will classify if a player is cheating or not.

This tutorial will walk the reader through this sample from beginning to end, starting with collecting, preparing, and submitting the dataset, building and deploying a classification model, and finally using that model to classify players as a potential cheater or honest gamer. If you'd like to follow along, you can sign up for one of our free Community accounts and use curl, [Postman](https://www.getpostman.com){:target="_blank"}, the [Nexosis Powershell client library](https://www.powershellgallery.com/packages/PSNexosisClient/2.1.0){:target="_blank"}, or one of our other [Client Libraries](http://docs.nexosis.com/clients/){:target="_blank"} in the language of your choice.

-----

## Gathering The Data

Acquiring publically available in-game player metrics is trivial since Valve's [Steam](https://en.wikipedia.org/wiki/Steam_(software)){:target="_blank"} platform collects and displays this information and makes it available via the [Steam API](https://developer.valvesoftware.com/wiki/Steam_Web_API){:target="_blank"}. Registered Steam users can be looked up via a unique Steam ID which can be used to retrieve public player info, in-game metrics, as well as if the player has been banned due to cheating detected via Valve Anti-Cheat (VAC). VAC has not always been very effective which is why we want to build our own model and see if we can do better.

> For more details on the topic of Valve Anti-Cheat's effectiveness, you can watch a talk that Ryan Sevey (Co-Founder/CEO) and I (Co-Founder/CTO) gave at DerbyCon back in 2014 titled [The Multibillion Dollar Industry That's Ignored](https://www.youtube.com/watch?v=xtvYUNF3JoQ&feature=youtu.be){:target="_blank"} which was key research and event leading up to the creation of Nexosis.

### All Class Types Must Be Represented In Training Dataset
Now that the Steam API has been identified as a source of the data, the last missing piece is how to find each class (cheating / not cheating) of player needed to build an effective classification model. The training dataset must be labeled, meaning each record must contain a column identifying cheaters and honest players. Additionally, we'll want to collect approximately an equal number of samples of both classes - the cheaters and honest players. We don't have to be exact though, because the Nexosis API will automatically balance the data set so it doesn't build a model biased towards one class or the other (unless given a parameter to build an unbalanced dataset).

There are many web sites dedicated to tracking Steam players that have been banned by the Valve Anti-Cheat, such as [vacbanned.com](https://vacbanned.com){:target="_blank"} and [vac-ban.com](http://vac-ban.com){:target="_blank"}. The data's not perfect though, as it's possible to get VACBanned for cheating in a different game, but it's the best we can do without having access to in-game metrics available internally to Valve, who is really in the best position to build the most effective models.

Finding a list of Steam ID's of users known to not be cheating is more difficult, because not all cheaters are detected by Valve's Anti-Cheat so we can't just rely on VACBanned set to 0 in the Steam API. Additionally, players will have a broad range of skill levels from beginner to professional level gamer. To find the cut-off point which should be the fine line between great to cheat we chose to collect data from players in the Professional Gaming League. These players are the best, but will still have human limitations, we need to make sure our model doesn't accidently classify them as cheaters either. Additionally, since these players are playing in competitive matches, they have an audience - since they are playing in the view of other professionals as well as the public, they are less likely to cheat for fear of getting caught and ruining their career.  There are plenty of web sites listing professional gamers, their statistics, as well as their Steam ID. 

Collecting a list of Steam ID's from those web sites is outside the scope of this tutorial, but there are various ways to go about that. Once you have a list of Steam IDs, the next step is to iterate over them all and query the Steam API for the game metrics and save them to a CSV to used to build a classification model.

### The Steam API

There are three main endpoints required from retrieving information collected to build a Classification Model. `GetUserStatsForGame` in order to get all the important metrics from the game as well as `GetOwnedGames` which we suspect is correlated to cheating as well - because [if you get VAC Banned you lose certain privileges on Steam making that Steam Account worthless for some things](https://support.steampowered.com/kb_article.php?ref=4044-qdhj-5691){:target="_blank"}. Additionally the API endpoint `GetPlayerBans` is used to make sure we don't accidently identify a player who was caught cheating as a non-cheater.

#### [GetUserStatsForGame](https://developer.valvesoftware.com/wiki/Steam_Web_API#GetUserStatsForGame_.28v0002.29){:target="_blank"}
`https://api.steampowered.com/ISteamUserStats/GetUserStatsForGame/v0002/`

Retrieving player stats from the Steam API is very simple. The API endpoint requires a Steam API key, a player's Steam ID to query, and the AppID of CS:GO.

<ul id="profileTabs" class="nav nav-tabs">
    <li class="active"><a href="#curl" data-toggle="tab">Curl</a></li>
    <li><a href="#powershell" data-toggle="tab">Powershell</a></li>
</ul>
<div class="tab-content">
    <div role="tabpanel" class="tab-pane" id="curl" class="tab-pane active" >
     <p>The following command assumes your Steam API Key is stored in an Environment Variable called <code>$steamApiKey</code>.</p><pre class="language-bash"><code class="language-bash code-toolbar">$ curl -X GET "https://api.steampowered.com/ISteamUserStats/GetUserStatsForGame/v0002/\
?steamid=76561198006920295&appid=730&key=$steamApiKey"</code></pre>Formatted HTTP Response:<pre class="language-json" style="max-height:30em;"><code class="language-json code-toolbar">{
        "playerstats": {
                "steamID": "76561198006920295",
                "gameName": "ValveTestApp260",
                "stats": [
                        {
                                "name": "total_kills",
                                "value": 299163
                        },
                        {
                                "name": "total_deaths",
                                "value": 246257
                        },
                        {
                                "name": "total_time_played",
                                "value": 14104846
                        },
                        {
                                "name": "total_planted_bombs",
                                "value": 8650
                        },
                        {
                                "name": "total_defused_bombs",
                                "value": 2784
                        },
                        {
                                "name": "total_wins",
                                "value": 103325
                        },
                        {
                                "name": "total_damage_done",
                                "value": 40678525
                        },
                        {
                                "name": "total_money_earned",
                                "value": 524966329
                        },
                        {
                                "name": "total_rescued_hostages",
                                "value": 3
                        },
                        {
                                "name": "total_kills_knife",
                                "value": 1800
                        },
                        {
                                "name": "total_kills_hegrenade",
                                "value": 692
                        },
                        {
                                "name": "total_kills_glock",
                                "value": 7797
                        },
                        {
                                "name": "total_kills_deagle",
                                "value": 11605
                        },
                        {
                                "name": "total_kills_elite",
                                "value": 262
                        },
                        {
                                "name": "total_kills_fiveseven",
                                "value": 1155
                        },
                        {
                                "name": "total_kills_xm1014",
                                "value": 87
                        },
                        {
                                "name": "total_kills_mac10",
                                "value": 1847
                        },
                        {
                                "name": "total_kills_ump45",
                                "value": 1520
                        },
                        {
                                "name": "total_kills_p90",
                                "value": 222
                        },
                        {
                                "name": "total_kills_awp",
                                "value": 28264
                        },
                        {
                                "name": "total_kills_ak47",
                                "value": 164005
                        },
                        {
                                "name": "total_kills_aug",
                                "value": 115
                        },
                        {
                                "name": "total_kills_famas",
                                "value": 2875
                        },
                        {
                                "name": "total_kills_g3sg1",
                                "value": 137
                        },
                        {
                                "name": "total_kills_m249",
                                "value": 48
                        },
                        {
                                "name": "total_kills_headshot",
                                "value": 207478
                        },
                        {
                                "name": "total_kills_enemy_weapon",
                                "value": 18806
                        },
                        {
                                "name": "total_wins_pistolround",
                                "value": 9216
                        },
                        {
                                "name": "total_decal_sprays",
                                "value": 5175
                        },
                        {
                                "name": "total_wins_map_cs_assault",
                                "value": 76
                        },
                        {
                                "name": "total_wins_map_cs_office",
                                "value": 18
                        },
                        {
                                "name": "total_wins_map_de_aztec",
                                "value": 18
                        },
                        {
                                "name": "total_wins_map_de_cbble",
                                "value": 7816
                        },
                        {
                                "name": "total_wins_map_de_dust2",
                                "value": 17493
                        },
                        {
                                "name": "total_wins_map_de_dust",
                                "value": 20
                        },
                        {
                                "name": "total_wins_map_de_inferno",
                                "value": 12928
                        },
                        {
                                "name": "total_wins_map_de_nuke",
                                "value": 5723
                        },
                        {
                                "name": "total_wins_map_de_train",
                                "value": 5583
                        },
                        {
                                "name": "total_weapons_donated",
                                "value": 20688
                        },
                        {
                                "name": "total_broken_windows",
                                "value": 1645
                        },
                        {
                                "name": "total_kills_enemy_blinded",
                                "value": 15342
                        },
                        {
                                "name": "total_kills_knife_fight",
                                "value": 1400
                        },
                        {
                                "name": "total_kills_against_zoomed_sniper",
                                "value": 6892
                        },
                        {
                                "name": "total_dominations",
                                "value": 1073
                        },
                        {
                                "name": "total_domination_overkills",
                                "value": 1976
                        },
                        {
                                "name": "total_revenges",
                                "value": 121
                        },
                        {
                                "name": "total_shots_hit",
                                "value": 747741
                        },
                        {
                                "name": "total_shots_fired",
                                "value": 4013872
                        },
                        {
                                "name": "total_rounds_played",
                                "value": 194578
                        },
                        {
                                "name": "total_shots_deagle",
                                "value": 87100
                        },
                        {
                                "name": "total_shots_glock",
                                "value": 137839
                        },
                        {
                                "name": "total_shots_elite",
                                "value": 6238
                        },
                        {
                                "name": "total_shots_fiveseven",
                                "value": 17231
                        },
                        {
                                "name": "total_shots_awp",
                                "value": 80754
                        },
                        {
                                "name": "total_shots_ak47",
                                "value": 2244368
                        },
                        {
                                "name": "total_shots_aug",
                                "value": 2132
                        },
                        {
                                "name": "total_shots_famas",
                                "value": 59804
                        },
                        {
                                "name": "total_shots_g3sg1",
                                "value": 1382
                        },
                        {
                                "name": "total_shots_p90",
                                "value": 8285
                        },
                        {
                                "name": "total_shots_mac10",
                                "value": 40621
                        },
                        {
                                "name": "total_shots_ump45",
                                "value": 26841
                        },
                        {
                                "name": "total_shots_xm1014",
                                "value": 4286
                        },
                        {
                                "name": "total_shots_m249",
                                "value": 1929
                        },
                        {
                                "name": "total_hits_deagle",
                                "value": 23833
                        },
                        {
                                "name": "total_hits_glock",
                                "value": 24548
                        },
                        {
                                "name": "total_hits_elite",
                                "value": 712
                        },
                        {
                                "name": "total_hits_fiveseven",
                                "value": 3588
                        },
                        {
                                "name": "total_hits_awp",
                                "value": 33579
                        },
                        {
                                "name": "total_hits_ak47",
                                "value": 373492
                        },
                        {
                                "name": "total_hits_aug",
                                "value": 344
                        },
                        {
                                "name": "total_hits_famas",
                                "value": 11769
                        },
                        {
                                "name": "total_hits_g3sg1",
                                "value": 246
                        },
                        {
                                "name": "total_hits_p90",
                                "value": 1021
                        },
                        {
                                "name": "total_hits_mac10",
                                "value": 8306
                        },
                        {
                                "name": "total_hits_ump45",
                                "value": 5472
                        },
                        {
                                "name": "total_hits_xm1014",
                                "value": 519
                        },
                        {
                                "name": "total_hits_m249",
                                "value": 270
                        },
                        {
                                "name": "total_rounds_map_cs_assault",
                                "value": 108
                        },
                        {
                                "name": "total_rounds_map_cs_office",
                                "value": 38
                        },
                        {
                                "name": "total_rounds_map_de_aztec",
                                "value": 21
                        },
                        {
                                "name": "total_rounds_map_de_cbble",
                                "value": 14599
                        },
                        {
                                "name": "total_rounds_map_de_dust2",
                                "value": 32834
                        },
                        {
                                "name": "total_rounds_map_de_dust",
                                "value": 45
                        },
                        {
                                "name": "total_rounds_map_de_inferno",
                                "value": 23697
                        },
                        {
                                "name": "total_rounds_map_de_nuke",
                                "value": 10646
                        },
                        {
                                "name": "total_rounds_map_de_train",
                                "value": 10592
                        },
                        {
                                "name": "last_match_t_wins",
                                "value": 16
                        },
                        {
                                "name": "last_match_ct_wins",
                                "value": 19
                        },
                        {
                                "name": "last_match_wins",
                                "value": 20
                        },
                        {
                                "name": "last_match_max_players",
                                "value": 13
                        },
                        {
                                "name": "last_match_kills",
                                "value": 27
                        },
                        {
                                "name": "last_match_deaths",
                                "value": 19
                        },
                        {
                                "name": "last_match_mvps",
                                "value": 7
                        },
                        {
                                "name": "last_match_favweapon_id",
                                "value": 7
                        },
                        {
                                "name": "last_match_favweapon_shots",
                                "value": 143
                        },
                        {
                                "name": "last_match_favweapon_hits",
                                "value": 32
                        },
                        {
                                "name": "last_match_favweapon_kills",
                                "value": 11
                        },
                        {
                                "name": "last_match_damage",
                                "value": 3655
                        },
                        {
                                "name": "last_match_money_spent",
                                "value": 90950
                        },
                        {
                                "name": "last_match_dominations",
                                "value": 0
                        },
                        {
                                "name": "last_match_revenges",
                                "value": 0
                        },
                        {
                                "name": "total_mvps",
                                "value": 28435
                        },
                        {
                                "name": "total_rounds_map_de_lake",
                                "value": 3
                        },
                        {
                                "name": "total_gun_game_rounds_won",
                                "value": 47
                        },
                        {
                                "name": "total_gun_game_rounds_played",
                                "value": 70
                        },
                        {
                                "name": "total_wins_map_ar_monastery",
                                "value": 1
                        },
                        {
                                "name": "total_rounds_map_ar_baggage",
                                "value": 1
                        },
                        {
                                "name": "total_wins_map_ar_baggage",
                                "value": 1
                        },
                        {
                                "name": "total_wins_map_de_lake",
                                "value": 2
                        },
                        {
                                "name": "total_matches_won",
                                "value": 2265
                        },
                        {
                                "name": "total_matches_played",
                                "value": 4263
                        },
                        {
                                "name": "total_gg_matches_won",
                                "value": 4
                        },
                        {
                                "name": "total_gg_matches_played",
                                "value": 75
                        },
                        {
                                "name": "total_progressive_matches_won",
                                "value": 4
                        },
                        {
                                "name": "total_contribution_score",
                                "value": 615399
                        },
                        {
                                "name": "last_match_contribution_score",
                                "value": 65
                        },
                        {
                                "name": "last_match_rounds",
                                "value": 35
                        },
                        {
                                "name": "total_kills_hkp2000",
                                "value": 17383
                        },
                        {
                                "name": "total_shots_hkp2000",
                                "value": 197073
                        },
                        {
                                "name": "total_hits_hkp2000",
                                "value": 43662
                        },
                        {
                                "name": "total_hits_p250",
                                "value": 20809
                        },
                        {
                                "name": "total_kills_p250",
                                "value": 6096
                        },
                        {
                                "name": "total_shots_p250",
                                "value": 100975
                        },
                        {
                                "name": "total_kills_sg556",
                                "value": 159
                        },
                        {
                                "name": "total_shots_sg556",
                                "value": 2514
                        },
                        {
                                "name": "total_hits_sg556",
                                "value": 417
                        },
                        {
                                "name": "total_hits_scar20",
                                "value": 635
                        },
                        {
                                "name": "total_kills_scar20",
                                "value": 345
                        },
                        {
                                "name": "total_shots_scar20",
                                "value": 3590
                        },
                        {
                                "name": "total_shots_ssg08",
                                "value": 3857
                        },
                        {
                                "name": "total_hits_ssg08",
                                "value": 1141
                        },
                        {
                                "name": "total_kills_ssg08",
                                "value": 545
                        },
                        {
                                "name": "total_shots_mp7",
                                "value": 6721
                        },
                        {
                                "name": "total_hits_mp7",
                                "value": 1352
                        },
                        {
                                "name": "total_kills_mp7",
                                "value": 291
                        },
                        {
                                "name": "total_kills_mp9",
                                "value": 1444
                        },
                        {
                                "name": "total_shots_mp9",
                                "value": 28406
                        },
                        {
                                "name": "total_hits_mp9",
                                "value": 6498
                        },
                        {
                                "name": "total_hits_nova",
                                "value": 1716
                        },
                        {
                                "name": "total_kills_nova",
                                "value": 259
                        },
                        {
                                "name": "total_shots_nova",
                                "value": 13906
                        },
                        {
                                "name": "total_hits_negev",
                                "value": 293
                        },
                        {
                                "name": "total_kills_negev",
                                "value": 87
                        },
                        {
                                "name": "total_shots_negev",
                                "value": 5412
                        },
                        {
                                "name": "total_shots_sawedoff",
                                "value": 2221
                        },
                        {
                                "name": "total_hits_sawedoff",
                                "value": 317
                        },
                        {
                                "name": "total_kills_sawedoff",
                                "value": 66
                        },
                        {
                                "name": "total_shots_bizon",
                                "value": 18579
                        },
                        {
                                "name": "total_hits_bizon",
                                "value": 1818
                        },
                        {
                                "name": "total_kills_bizon",
                                "value": 434
                        },
                        {
                                "name": "total_kills_tec9",
                                "value": 1651
                        },
                        {
                                "name": "total_shots_tec9",
                                "value": 29949
                        },
                        {
                                "name": "total_hits_tec9",
                                "value": 5126
                        },
                        {
                                "name": "total_shots_mag7",
                                "value": 10689
                        },
                        {
                                "name": "total_hits_mag7",
                                "value": 1730
                        },
                        {
                                "name": "total_kills_mag7",
                                "value": 270
                        },
                        {
                                "name": "total_gun_game_contribution_score",
                                "value": 296
                        },
                        {
                                "name": "last_match_gg_contribution_score",
                                "value": 0
                        },
                        {
                                "name": "total_kills_m4a1",
                                "value": 46294
                        },
                        {
                                "name": "total_kills_galilar",
                                "value": 1148
                        },
                        {
                                "name": "total_kills_molotov",
                                "value": 236
                        },
                        {
                                "name": "total_kills_taser",
                                "value": 22
                        },
                        {
                                "name": "total_shots_m4a1",
                                "value": 835904
                        },
                        {
                                "name": "total_shots_galilar",
                                "value": 34978
                        },
                        {
                                "name": "total_shots_taser",
                                "value": 288
                        },
                        {
                                "name": "total_hits_m4a1",
                                "value": 174252
                        },
                        {
                                "name": "total_hits_galilar",
                                "value": 4440
                        },
                        {
                                "name": "total_rounds_map_ar_monastery",
                                "value": 2
                        },
                        {
                                "name": "total_matches_won_train",
                                "value": 78
                        },
                        {
                                "name": "total_matches_won_baggage",
                                "value": 1
                        },
                        {
                                "name": "total_matches_won_lake",
                                "value": 2
                        },
                        {
                                "name": "GI.lesson.csgo_instr_explain_buymenu",
                                "value": 16
                        },
                        {
                                "name": "GI.lesson.csgo_instr_explain_buyarmor",
                                "value": 16
                        },
                        {
                                "name": "GI.lesson.csgo_instr_explain_plant_bomb",
                                "value": 16
                        },
                        {
                                "name": "GI.lesson.csgo_instr_explain_bomb_carrier",
                                "value": 1
                        },
                        {
                                "name": "GI.lesson.bomb_sites_A",
                                "value": 1
                        },
                        {
                                "name": "GI.lesson.defuse_planted_bomb",
                                "value": 1
                        },
                        {
                                "name": "GI.lesson.csgo_instr_explain_follow_bomber",
                                "value": 1
                        },
                        {
                                "name": "GI.lesson.csgo_instr_explain_pickup_bomb",
                                "value": 1
                        },
                        {
                                "name": "GI.lesson.csgo_instr_explain_prevent_bomb_pickup",
                                "value": 1
                        },
                        {
                                "name": "GI.lesson.Csgo_cycle_weapons_kb",
                                "value": 16
                        },
                        {
                                "name": "GI.lesson.csgo_instr_explain_zoom",
                                "value": 16
                        },
                        {
                                "name": "GI.lesson.csgo_instr_explain_silencer",
                                "value": 16
                        },
                        {
                                "name": "GI.lesson.csgo_instr_explain_reload",
                                "value": 17
                        },
                        {
                                "name": "GI.lesson.tr_explain_plant_bomb",
                                "value": 0
                        },
                        {
                                "name": "GI.lesson.bomb_sites_B",
                                "value": 1
                        },
                        {
                                "name": "GI.lesson.version_number",
                                "value": 16
                        },
                        {
                                "name": "GI.lesson.find_planted_bomb",
                                "value": 1
                        },
                        {
                                "name": "GI.lesson.csgo_instr_explain_inspect",
                                "value": 32
                        }
                ]
                ,
                "achievements": [
                        {
                                "name": "WIN_BOMB_PLANT",
                                "achieved": 1
                        },
                        {
                                "name": "BOMB_PLANT_LOW",
                                "achieved": 1
                        },
                        {
                                "name": "BOMB_DEFUSE_LOW",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_ENEMY_LOW",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_ENEMY_MED",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_ENEMY_HIGH",
                                "achieved": 1
                        },
                        {
                                "name": "BOMB_DEFUSE_CLOSE_CALL",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_BOMB_DEFUSER",
                                "achieved": 1
                        },
                        {
                                "name": "WIN_BOMB_DEFUSE",
                                "achieved": 1
                        },
                        {
                                "name": "BOMB_PLANT_IN_25_SECONDS",
                                "achieved": 1
                        },
                        {
                                "name": "WIN_ROUNDS_LOW",
                                "achieved": 1
                        },
                        {
                                "name": "WIN_ROUNDS_MED",
                                "achieved": 1
                        },
                        {
                                "name": "WIN_ROUNDS_HIGH",
                                "achieved": 1
                        },
                        {
                                "name": "GIVE_DAMAGE_LOW",
                                "achieved": 1
                        },
                        {
                                "name": "GIVE_DAMAGE_MED",
                                "achieved": 1
                        },
                        {
                                "name": "GIVE_DAMAGE_HIGH",
                                "achieved": 1
                        },
                        {
                                "name": "KILLING_SPREE",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_WITH_OWN_GUN",
                                "achieved": 1
                        },
                        {
                                "name": "RESCUE_ALL_HOSTAGES",
                                "achieved": 1
                        },
                        {
                                "name": "FAST_HOSTAGE_RESCUE",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_TWO_WITH_ONE_SHOT",
                                "achieved": 1
                        },
                        {
                                "name": "EARN_MONEY_LOW",
                                "achieved": 1
                        },
                        {
                                "name": "EARN_MONEY_MED",
                                "achieved": 1
                        },
                        {
                                "name": "EARN_MONEY_HIGH",
                                "achieved": 1
                        },
                        {
                                "name": "DEAD_GRENADE_KILL",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_ENEMY_DEAGLE",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_ENEMY_GLOCK",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_ENEMY_ELITE",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_ENEMY_FIVESEVEN",
                                "achieved": 1
                        },
                        {
                                "name": "META_PISTOL",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_ENEMY_AWP",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_ENEMY_AK47",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_ENEMY_M4A1",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_ENEMY_FAMAS",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_ENEMY_G3SG1",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_ENEMY_MAC10",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_ENEMY_UMP45",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_ENEMY_HEGRENADE",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_ENEMY_KNIFE",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_ENEMY_TEAM",
                                "achieved": 1
                        },
                        {
                                "name": "KILLS_WITH_MULTIPLE_GUNS",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_HOSTAGE_RESCUER",
                                "achieved": 1
                        },
                        {
                                "name": "LAST_PLAYER_ALIVE",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_ENEMY_LAST_BULLET",
                                "achieved": 1
                        },
                        {
                                "name": "KILLING_SPREE_ENDER",
                                "achieved": 1
                        },
                        {
                                "name": "HEADSHOTS",
                                "achieved": 1
                        },
                        {
                                "name": "DAMAGE_NO_KILL",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_LOW_DAMAGE",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_ENEMY_RELOADING",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_ENEMY_BLINDED",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_ENEMIES_WHILE_BLIND",
                                "achieved": 1
                        },
                        {
                                "name": "KILLS_ENEMY_WEAPON",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_WITH_EVERY_WEAPON",
                                "achieved": 1
                        },
                        {
                                "name": "SURVIVE_GRENADE",
                                "achieved": 1
                        },
                        {
                                "name": "WIN_KNIFE_FIGHTS_LOW",
                                "achieved": 1
                        },
                        {
                                "name": "WIN_KNIFE_FIGHTS_HIGH",
                                "achieved": 1
                        },
                        {
                                "name": "HIP_SHOT",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_SNIPER_WITH_SNIPER",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_SNIPER_WITH_KNIFE",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_SNIPERS",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_WHEN_AT_LOW_HEALTH",
                                "achieved": 1
                        },
                        {
                                "name": "GRENADE_MULTIKILL",
                                "achieved": 1
                        },
                        {
                                "name": "PISTOL_ROUND_KNIFE_KILL",
                                "achieved": 1
                        },
                        {
                                "name": "FAST_ROUND_WIN",
                                "achieved": 1
                        },
                        {
                                "name": "WIN_PISTOLROUNDS_LOW",
                                "achieved": 1
                        },
                        {
                                "name": "WIN_PISTOLROUNDS_MED",
                                "achieved": 1
                        },
                        {
                                "name": "WIN_PISTOLROUNDS_HIGH",
                                "achieved": 1
                        },
                        {
                                "name": "GOOSE_CHASE",
                                "achieved": 1
                        },
                        {
                                "name": "WIN_BOMB_PLANT_AFTER_RECOVERY",
                                "achieved": 1
                        },
                        {
                                "name": "SURVIVE_MANY_ATTACKS",
                                "achieved": 1
                        },
                        {
                                "name": "LOSSLESS_EXTERMINATION",
                                "achieved": 1
                        },
                        {
                                "name": "FLAWLESS_VICTORY",
                                "achieved": 1
                        },
                        {
                                "name": "WIN_DUAL_DUEL",
                                "achieved": 1
                        },
                        {
                                "name": "UNSTOPPABLE_FORCE",
                                "achieved": 1
                        },
                        {
                                "name": "IMMOVABLE_OBJECT",
                                "achieved": 1
                        },
                        {
                                "name": "HEADSHOTS_IN_ROUND",
                                "achieved": 1
                        },
                        {
                                "name": "WIN_MAP_DE_DUST2",
                                "achieved": 1
                        },
                        {
                                "name": "WIN_MAP_DE_INFERNO",
                                "achieved": 1
                        },
                        {
                                "name": "WIN_MAP_DE_NUKE",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_WHILE_IN_AIR",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_ENEMY_IN_AIR",
                                "achieved": 1
                        },
                        {
                                "name": "KILLER_AND_ENEMY_IN_AIR",
                                "achieved": 1
                        },
                        {
                                "name": "SILENT_WIN",
                                "achieved": 1
                        },
                        {
                                "name": "BLOODLESS_VICTORY",
                                "achieved": 1
                        },
                        {
                                "name": "DONATE_WEAPONS",
                                "achieved": 1
                        },
                        {
                                "name": "WIN_ROUNDS_WITHOUT_BUYING",
                                "achieved": 1
                        },
                        {
                                "name": "DEFUSE_DEFENSE",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_BOMB_PICKUP",
                                "achieved": 1
                        },
                        {
                                "name": "DOMINATIONS_LOW",
                                "achieved": 1
                        },
                        {
                                "name": "DOMINATIONS_HIGH",
                                "achieved": 1
                        },
                        {
                                "name": "DOMINATION_OVERKILLS_LOW",
                                "achieved": 1
                        },
                        {
                                "name": "DOMINATION_OVERKILLS_HIGH",
                                "achieved": 1
                        },
                        {
                                "name": "REVENGES_LOW",
                                "achieved": 1
                        },
                        {
                                "name": "REVENGES_HIGH",
                                "achieved": 1
                        },
                        {
                                "name": "CONCURRENT_DOMINATIONS",
                                "achieved": 1
                        },
                        {
                                "name": "DOMINATION_OVERKILLS_MATCH",
                                "achieved": 1
                        },
                        {
                                "name": "EXTENDED_DOMINATION",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_ENEMIES_WHILE_BLIND_HARD",
                                "achieved": 1
                        },
                        {
                                "name": "CAUSE_FRIENDLY_FIRE_WITH_FLASHBANG",
                                "achieved": 1
                        },
                        {
                                "name": "AVENGE_FRIEND",
                                "achieved": 1
                        },
                        {
                                "name": "WIN_GUN_GAME_ROUNDS_LOW",
                                "achieved": 1
                        },
                        {
                                "name": "GUN_GAME_FIRST_KILL",
                                "achieved": 1
                        },
                        {
                                "name": "BASE_SCAMPER",
                                "achieved": 1
                        },
                        {
                                "name": "BORN_READY",
                                "achieved": 1
                        },
                        {
                                "name": "STILL_ALIVE",
                                "achieved": 1
                        },
                        {
                                "name": "MEDALIST",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_ENEMY_BIZON",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_ENEMY_TEC9",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_ENEMY_TASER",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_ENEMY_HKP2000",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_ENEMY_P250",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_ENEMY_SCAR20",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_ENEMY_SG556",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_ENEMY_SSG08",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_ENEMY_MP7",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_ENEMY_MP9",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_ENEMY_MAG7",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_ENEMY_SAWEDOFF",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_ENEMY_NOVA",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_ENEMY_MOLOTOV",
                                "achieved": 1
                        },
                        {
                                "name": "WIN_MAP_DE_TRAIN",
                                "achieved": 1
                        },
                        {
                                "name": "KILL_ENEMY_GALILAR",
                                "achieved": 1
                        }
                ]

        }
}</code>
      </pre>
    </div>
     <div role="tabpanel" class="tab-pane" id="powershell">
      <p>The following command assumes your Steam API Key is stored in an Environment Variable called <code>$steamApiKey</code>.</p><pre class="language-powershell"><code class="language-powershell code-toolbar">PS> $results = Invoke-RestMethod -Method Get `
        -Uri $("https://api.steampowered.com/ISteamUserStats/GetUserStatsForGame" + `
               "/v0002/?steamid=76561198006920295&appid=730&key=$env:steamApiKey")
PS> $results.playerstats.stats</code></pre><pre class="language-powershell" style="max-height:30em;"><code class="language-powershell code-toolbar">name                                                 value
----                                                 -----
total_kills                                         298765
total_deaths                                        245897
total_time_played                                 14070703
total_planted_bombs                                   8635
total_defused_bombs                                   2778
total_wins                                          103096
total_damage_done                                 40624879
total_money_earned                               523721979
total_rescued_hostages                                   3
total_kills_knife                                     1799
total_kills_hegrenade                                  692
total_kills_glock                                     7784
total_kills_deagle                                   11592
total_kills_elite                                      262
total_kills_fiveseven                                 1155
total_kills_xm1014                                      87
total_kills_mac10                                     1842
total_kills_ump45                                     1518
total_kills_p90                                        222
total_kills_awp                                      28215
total_kills_ak47                                    163808
total_kills_aug                                        115
total_kills_famas                                     2865
total_kills_g3sg1                                      137
total_kills_m249                                        48
total_kills_headshot                                207235
total_kills_enemy_weapon                             18740
total_wins_pistolround                                9199
total_decal_sprays                                    5175
total_wins_map_cs_assault                               76
total_wins_map_cs_office                                18
total_wins_map_de_aztec                                 18
total_wins_map_de_cbble                               7715
total_wins_map_de_dust2                              17492
total_wins_map_de_dust                                  20
total_wins_map_de_inferno                            12858
total_wins_map_de_nuke                                5723  
total_wins_map_de_train                               5583
total_weapons_donated                                20622
total_broken_windows                                  1645
total_kills_enemy_blinded                            15311
total_kills_knife_fight                               1400
total_kills_against_zoomed_sniper                     6885
total_dominations                                     1073
total_domination_overkills                            1976
total_revenges                                         121
total_shots_hit                                     746743
total_shots_fired                                  4008615
total_rounds_played                                 194143
total_shots_deagle                                   86984
total_shots_glock                                   137623
total_shots_elite                                     6238
total_shots_fiveseven                                17231
total_shots_awp                                      80631
total_shots_ak47                                   2241608
total_shots_aug                                       2132
total_shots_famas                                    59683
total_shots_g3sg1                                     1382
total_shots_p90                                       8285
total_shots_mac10                                    40436
total_shots_ump45                                    26792
total_shots_xm1014                                    4286
total_shots_m249                                      1929
total_hits_deagle                                    23807
total_hits_glock                                     24509
total_hits_elite                                       712
total_hits_fiveseven                                  3588
total_hits_awp                                       33526
total_hits_ak47                                     373031
total_hits_aug                                         344
total_hits_famas                                     11740
total_hits_g3sg1                                       246
total_hits_p90                                        1021
total_hits_mac10                                      8275
total_hits_ump45                                      5456
total_hits_xm1014                                      519
total_hits_m249                                        270
total_rounds_map_cs_assault                            108
total_rounds_map_cs_office                              38
total_rounds_map_de_aztec                               21
total_rounds_map_de_cbble                            14393
total_rounds_map_de_dust2                            32830
total_rounds_map_de_dust                                45
total_rounds_map_de_inferno                          23576
total_rounds_map_de_nuke                             10646
total_rounds_map_de_train                            10592
last_match_t_wins                                       21
last_match_ct_wins                                      10
last_match_wins                                         15
last_match_max_players                                  10
last_match_kills                                        18
last_match_deaths                                       25
last_match_mvps                                          2
last_match_favweapon_id                                  7
last_match_favweapon_shots                             201
last_match_favweapon_hits                               17
last_match_favweapon_kills                               7
last_match_damage                                     2551
last_match_money_spent                               86850
last_match_dominations                                   0
last_match_revenges                                      0
total_mvps                                           28385
total_rounds_map_de_lake                                 3
total_gun_game_rounds_won                               47
total_gun_game_rounds_played                            70
total_wins_map_ar_monastery                              1
total_rounds_map_ar_baggage                              1
total_wins_map_ar_baggage                                1
total_wins_map_de_lake                                   2
total_matches_won                                     2261
total_matches_played                                  4257
total_gg_matches_won                                     4
total_gg_matches_played                                 75
total_progressive_matches_won                            4
total_contribution_score                            614660
last_match_contribution_score                           50
last_match_rounds                                       31
total_kills_hkp2000                                  17359
total_shots_hkp2000                                 196751
total_hits_hkp2000                                   43600
total_hits_p250                                      20760
total_kills_p250                                      6081
total_shots_p250                                    100776
total_kills_sg556                                      159
total_shots_sg556                                     2514
total_hits_sg556                                       417
total_hits_scar20                                      635
total_kills_scar20                                     345
total_shots_scar20                                    3590
total_shots_ssg08                                     3841
total_hits_ssg08                                      1137
total_kills_ssg08                                      541
total_shots_mp7                                       6721
total_hits_mp7                                        1352
total_kills_mp7                                        291
total_kills_mp9                                       1438
total_shots_mp9                                      28317
total_hits_mp9                                        6475
total_hits_nova                                       1716
total_kills_nova                                       259
total_shots_nova                                     13906
total_hits_negev                                       293
total_kills_negev                                       87
total_shots_negev                                     5412
total_shots_sawedoff                                  2221
total_hits_sawedoff                                    317
total_kills_sawedoff                                    66
total_shots_bizon                                    18579
total_hits_bizon                                      1818
total_kills_bizon                                      434
total_kills_tec9                                      1651
total_shots_tec9                                     29949
total_hits_tec9                                       5126
total_shots_mag7                                     10689
total_hits_mag7                                       1730
total_kills_mag7                                       270
total_gun_game_contribution_score                      296
last_match_gg_contribution_score                         0
total_kills_m4a1                                     46235
total_kills_galilar                                   1148
total_kills_molotov                                    236
total_kills_taser                                       22
total_shots_m4a1                                    834880
total_shots_galilar                                  34943
total_shots_taser                                      286
total_hits_m4a1                                     174043
total_hits_galilar                                    4439
total_rounds_map_ar_monastery                            2
total_matches_won_train                                 78
total_matches_won_baggage                                1
total_matches_won_lake                                   2
GI.lesson.csgo_instr_explain_buymenu                    16
GI.lesson.csgo_instr_explain_buyarmor                   16
GI.lesson.csgo_instr_explain_plant_bomb                 16
GI.lesson.csgo_instr_explain_bomb_carrier                1
GI.lesson.bomb_sites_A                                   1
GI.lesson.defuse_planted_bomb                            1
GI.lesson.csgo_instr_explain_follow_bomber               1
GI.lesson.csgo_instr_explain_pickup_bomb                 1
GI.lesson.csgo_instr_explain_prevent_bomb_pickup         1
GI.lesson.Csgo_cycle_weapons_kb                         16
GI.lesson.csgo_instr_explain_zoom                       16
GI.lesson.csgo_instr_explain_silencer                   16
GI.lesson.csgo_instr_explain_reload                     17
GI.lesson.tr_explain_plant_bomb                          0
GI.lesson.bomb_sites_B                                   1
GI.lesson.version_number                                16
GI.lesson.find_planted_bomb                              1
GI.lesson.csgo_instr_explain_inspect                    32
          </code>
      </pre>
    </div>
</div>

As you can see above, Counter-strike tracks approximately 190 public metrics, many of which might not be helpful for detecting cheaters. It will be important to decide which ones to eliminate and which ones to combine to create a new, more meaningful metrics. More on that in the next section.

#### [GetOwnedGames](https://developer.valvesoftware.com/wiki/Steam_Web_API#GetOwnedGames_.28v0001.29){:target="_blank"}
`https://api.steampowered.com/IPlayerService/GetOwnedGames/v0001/`

Getting caught cheating results in a ban on a user's Steam Profile so many cheaters won't risk getting caught using their primary profile since it could impact other games they own. This type of account is called a 'Smurf account.' Using the number of games a player owns as one of our features seems like a good idea. However, more recently Steam requires a cell phone number linked to profiles and a ban will impact accounts sharing the same number. While a nice additional step, there are some ways around this (like using your spouse or parents phone number for instance).

<ul id="profileTabs" class="nav nav-tabs">
    <li class="active"><a href="#curlOwned" data-toggle="tab">Curl</a></li>
    <li><a href="#powershellOwned" data-toggle="tab">Powershell</a></li>
</ul>
<div class="tab-content">
    <div role="tabpanel" class="tab-pane active" id="curlOwned">
     <p>The following command assumes your Steam API Key is stored in an Environment Variable called <code>$steamApiKey</code>.</p><pre class="language-bash"><code class="language-bash code-toolbar">$ curl -X GET "https://api.steampowered.com/IPlayerService/GetOwnedGames/v0001/?\
key=$steamApiKey&steamid=76561198006920295&format=json"</code></pre>Formatted HTTP Response:<pre class="language-json" style="max-height:30em;"><code class="language-json code-toolbar">{
    "response": {
        "game_count": 76,
        "games": [
                {
                        "appid": 10,
                        "playtime_forever": 17726
                },
                {
                        "appid": 80,
                        "playtime_forever": 216
                },
                {
                        "appid": 100,
                        "playtime_forever": 0
                },
                {
                        "appid": 240,
                        "playtime_forever": 134014
                },
                {
                        "appid": 7200,
                        "playtime_forever": 59
                },
                {
                        "appid": 4540,
                        "playtime_forever": 3
                },
                {
                        "appid": 4550,
                        "playtime_forever": 77
                },
                {
                        "appid": 475150,
                        "playtime_forever": 0
                },
                {
                        "appid": 10150,
                        "playtime_forever": 270
                },
                {
                        "appid": 29670,
                        "playtime_forever": 0
                },
                {
                        "appid": 10180,
                        "playtime_forever": 0
                },
                {
                        "appid": 10190,
                        "playtime_forever": 725
                },
                {
                        "appid": 550,
                        "playtime_forever": 0
                },
                {
                        "appid": 223530,
                        "playtime_forever": 0
                },
                {
                        "appid": 18820,
                        "playtime_forever": 2574
                },
                {
                        "appid": 24960,
                        "playtime_forever": 2052
                },
                {
                        "appid": 205790,
                        "playtime_forever": 0
                },
                {
                        "appid": 400,
                        "playtime_forever": 0
                },
                {
                        "appid": 420,
                        "playtime_forever": 0
                },
                {
                        "appid": 9420,
                        "playtime_forever": 55
                },
                {
                        "appid": 105450,
                        "playtime_forever": 10846
                },
                {
                        "appid": 730,
                        "playtime_2weeks": 3596,
                        "playtime_forever": 390898
                },
                {
                        "appid": 49520,
                        "playtime_forever": 140
                },
                {
                        "appid": 11020,
                        "playtime_forever": 462
                },
                {
                        "appid": 109600,
                        "playtime_forever": 0
                },
                {
                        "appid": 208090,
                        "playtime_forever": 0
                },
                {
                        "appid": 218230,
                        "playtime_forever": 0
                },
                {
                        "appid": 230410,
                        "playtime_forever": 0
                },
                {
                        "appid": 238260,
                        "playtime_forever": 0
                },
                {
                        "appid": 238960,
                        "playtime_forever": 270
                },
                {
                        "appid": 272350,
                        "playtime_forever": 0
                },
                {
                        "appid": 209870,
                        "playtime_forever": 0
                },
                {
                        "appid": 237310,
                        "playtime_forever": 0
                },
                {
                        "appid": 226320,
                        "playtime_forever": 0
                },
                {
                        "appid": 39140,
                        "playtime_forever": 2816
                },
                {
                        "appid": 233840,
                        "playtime_forever": 1135
                },
                {
                        "appid": 207140,
                        "playtime_forever": 1198
                },
                {
                        "appid": 252950,
                        "playtime_forever": 0
                },
                {
                        "appid": 261430,
                        "playtime_forever": 155
                },
                {
                        "appid": 266840,
                        "playtime_forever": 117
                },
                {
                        "appid": 39150,
                        "playtime_forever": 15
                },
                {
                        "appid": 273110,
                        "playtime_forever": 0
                },
                {
                        "appid": 282440,
                        "playtime_forever": 0
                },
                {
                        "appid": 287700,
                        "playtime_forever": 1521
                },
                {
                        "appid": 290140,
                        "playtime_forever": 91
                },
                {
                        "appid": 295110,
                        "playtime_forever": 1893
                },
                {
                        "appid": 362300,
                        "playtime_forever": 0
                },
                {
                        "appid": 433850,
                        "playtime_forever": 2626
                },
                {
                        "appid": 439700,
                        "playtime_forever": 0
                },
                {
                        "appid": 222900,
                        "playtime_forever": 0
                },
                {
                        "appid": 292120,
                        "playtime_forever": 150
                },
                {
                        "appid": 304930,
                        "playtime_forever": 0
                },
                {
                        "appid": 319910,
                        "playtime_forever": 0
                },
                {
                        "appid": 221380,
                        "playtime_forever": 3490
                },
                {
                        "appid": 323370,
                        "playtime_forever": 0
                },
                {
                        "appid": 341800,
                        "playtime_forever": 145
                },
                {
                        "appid": 346110,
                        "playtime_forever": 0
                },
                {
                        "appid": 407530,
                        "playtime_forever": 0
                },
                {
                        "appid": 346900,
                        "playtime_forever": 0
                },
                {
                        "appid": 351510,
                        "playtime_forever": 37
                },
                {
                        "appid": 355950,
                        "playtime_forever": 0
                },
                {
                        "appid": 271590,
                        "playtime_forever": 4435
                },
                {
                        "appid": 359870,
                        "playtime_forever": 1388
                },
                {
                        "appid": 363970,
                        "playtime_forever": 0
                },
                {
                        "appid": 311210,
                        "playtime_forever": 641
                },
                {
                        "appid": 455130,
                        "playtime_forever": 0
                },
                {
                        "appid": 380600,
                        "playtime_forever": 0
                },
                {
                        "appid": 291550,
                        "playtime_forever": 0
                },
                {
                        "appid": 359550,
                        "forever": 450
                },
                {
                        "appid": 623990,
                        "playtime_forever": 0
                },
                {
                        "appid": 431240,
                        "playtime_forever": 0
                },
                {
                        "appid": 440900,
                        "playtime_forever": 0
                },
                {
                        "appid": 444090,
                        "playtime_forever": 0
                },
                {
                        "appid": 596350,
                        "playtime_forever": 0
                },
                {
                        "appid": 365590,
                        "playtime_forever": 2367
                },
                {
                        "appid": 524440,
                        "playtime_forever": 0
                }
        ]
    }
}</code></pre>
    </div>
    <div role="tabpanel" class="tab-pane" id="powershellOwned">
      <p>The following command assumes your Steam API Key is stored in an Environment Variable called <code>$steamApiKey</code>.</p>
      <pre class="language-powershell"><code class="language-powershell code-toolbar">PS> $playerSummary = Invoke-RestMethod -Method Get `
	 -Uri "https://api.steampowered.com/IPlayerService/GetOwnedGames/v0001/?key=$env:steamApiKey&steamid=76561198006920295&format=json"
PS> $playerSummary.response</code></pre><pre class="language-powershell" style="max-height:30em;"><code class="language-powershell code-toolbar">
game_count games                                                                                                                                               
----------	-----                                         
76 			{@{appid=10; playtime_forever=17726}, @{appid=80; playtime_forever=216}, @{appid=100; playtime_forever=0}, @{appid=240; playtime_forever=134014}...}
          </code>
      </pre>
    </div>
</div>

#### [GetPlayerBans](https://developer.valvesoftware.com/wiki/Steam_Web_API#GetPlayerBans_.28v1.29){:target="_blank"}
`https://api.steampowered.com/ISteamUser/GetPlayerBans/v1/`
Player Bans endpoint contains the "VACBanned" property that we will use to label the cheater dataset.

<ul id="profileTabs" class="nav nav-tabs">
    <li class="active"><a href="#curlGetPlayerBans" data-toggle="tab">Curl</a></li>
    <li><a href="#powershellGetPlayerBans" data-toggle="tab">Powershell</a></li>
</ul>
<div class="tab-content">
    <div role="tabpanel" class="tab-pane active" id="curlGetPlayerBans">
     <p>The following command assumes your Steam API Key is stored in an Environment Variable called <code>$steamApiKey</code>.</p><pre class="language-bash"><code class="language-bash code-toolbar">$ curl -X GET "http://api.steampowered.com/ISteamUser/GetPlayerBans/v1/?key=$steamApiKey\
&steamids=76561198006920295"</code></pre>Formatted HTTP Response:<pre class="language-json" style="max-height:30em;"><code class="language-json code-toolbar">{
  "players": [
                {
                        "SteamId": "76561198006920295",
                        "CommunityBanned": false,
                        "VACBanned": false,
                        "NumberOfVACBans": 0,
                        "DaysSinceLastBan": 0,
                        "NumberOfGameBans": 0,
                        "EconomyBan": "none"
                }
        ]

}
</code></pre>
    </div>
    <div role="tabpanel" class="tab-pane" id="powershellGetPlayerBans">
      <p>The following command assumes your Steam API Key is stored in an Environment Variable called <code>$steamApiKey</code>.</p>
      <pre class="language-powershell"><code class="language-powershell code-toolbar">PS> $playerSummary = Invoke-RestMethod -Method Get `
	 -Uri "http://api.steampowered.com/ISteamUser/GetPlayerBans/v1/?key=$env:steamApiKey&steamids=76561198006920295"
PS> $playerBans.players[0]

teamId          : 76561198006920295
CommunityBanned  : False
VACBanned        : False
NumberOfVACBans  : 0
DaysSinceLastBan : 0
NumberOfGameBans : 0
EconomyBan       : none
</code>
      </pre>
    </div>
</div>

#### [GetPlayerSummaries](https://developer.valvesoftware.com/wiki/Steam_Web_API#GetPlayerSummaries_.28v0002.29){:target="_blank"}
`https://api.steampowered.com/ISteamUser/GetPlayerSummaries/v0002/`

The Players Summary information isn't required, but is useful for querying to see if the Player Profile is public before attempting to move on and pull down the in-game metrics. If their profile is not public, we cannot get the stats or other information needed so we should just ignore those.

As long as the parameter CommunityVisibilityState is set to 3, we can retrieve their in-game stats.

* <b><code>CommunityVisibilityState</code></b>
  * <code>1</code> - Private, stats are hidden.
  * <code>2</code> - FriendsOnly, stats are only visible to their friends.
  * <code>3</code> - Public, stats are visible to anyone.

<ul id="profileTabs" class="nav nav-tabs">
    <li class="active"><a href="#curlGetPlayerSummaries" data-toggle="tab">Curl</a></li>
    <li><a href="#powershellGetPlayerSummaries" data-toggle="tab">Powershell</a></li>
</ul>
<div class="tab-content">
    <div role="tabpanel" class="tab-pane active" id="curlGetPlayerSummaries">
    <p>The following command assumes your Steam API Key is stored in an Environment Variable called <code>$steamApiKey</code>.</p><pre class="language-bash"><code class="language-bash code-toolbar">$ curl -X GET "http://api.steampowered.com/ISteamUser/GetPlayerSummaries/v0002/\
?key=$steamApiKey&steamids=76561198006920295"</code></pre>Formatted HTTP Response:<pre class="language-json" style="max-height:30em;"><code class="language-json code-toolbar">{
  "response": {
    "players": [
      {
        "steamid": "76561198006920295",
        "communityvisibilitystate": 3,
        "profilestate": 1,
        "personaname": "FT _|",
        "lastlogoff": 1515441121,
        "commentpermission": 1,
        "profileurl": "http://steamcommunity.com/profiles/76561198006920295/",
        "avatar": "https://steamcdn-a.akamaihd.net/steamcommunity/public/images/avatars/7f/7fee3ec36dcb13c54a93955c6f0ef570ae4d52dc.jpg",
        "avatarmedium": "https://steamcdn-a.akamaihd.net/steamcommunity/public/images/avatars/7f/7fee3ec36dcb13c54a93955c6f0ef570ae4d52dc_medium.jpg",
        "avatarfull": "https://steamcdn-a.akamaihd.net/steamcommunity/public/images/avatars/7f/7fee3ec36dcb13c54a93955c6f0ef570ae4d52dc_full.jpg",
        "personastate": 0,
        "realname": "Papillon Richard",
        "primaryclanid": "103582791431818521",
        "timecreated": 1235668712,
        "personastateflags": 0,
        "loccountrycode": "FR",
        "locstatecode": "A8"
      }
    ]
  }
}</code></pre>
    </div>
    <div role="tabpanel" class="tab-pane" id="powershellGetPlayerSummaries">
      <p>The following command assumes your Steam API Key is stored in an Environment Variable called <code>$steamApiKey</code>.</p>
      <pre class="language-powershell"><code class="language-powershell code-toolbar">PS> $playerSummary = Invoke-RestMethod -Method Get `
    -Uri "http://api.steampowered.com/ISteamUser/GetPlayerSummaries/v0002/?key=$env:steamApiKey&steamids=76561198006920295"
PS> $playerSummary.response.players[0]

steamid                  : 76561198006920295
communityvisibilitystate : 3
profilestate             : 1
personaname              : shox
lastlogoff               : 1512697001
commentpermission        : 1
profileurl               : http://steamcommunity.com/profiles/76561198006920295/
avatar                   : https://steamcdn-a.akamaihd.net/steamcommunity/public/images/avatars/7f/7fee3ec36dcb13c54a93955c6f0ef570ae4d52dc.jpg
avatarmedium             : https://steamcdn-a.akamaihd.net/steamcommunity/public/images/avatars/7f/7fee3ec36dcb13c54a93955c6f0ef570ae4d52dc_medium.jpg
avatarfull               : https://steamcdn-a.akamaihd.net/steamcommunity/public/images/avatars/7f/7fee3ec36dcb13c54a93955c6f0ef570ae4d52dc_full.jpg
personastate             : 3
realname                 : Papillon Richard
primaryclanid            : 103582791431818521
timecreated              : 1235668712
personastateflags        : 0
loccountrycode           : FR
locstatecode             : A8
  </code>
</pre>
    </div>
</div>
## Understanding and Preparing The Data

Based on the amount of data available on a Player based on just the 4 Steam API endpoints - there is a choice of over 250 pieces of information. One could blindly jam every single one of these data-points in the Nexosis API, cross our fingers and hope it finds something - but if there's no correlation between each column and the target, we'll end up with a useless model.

This means putting some thought into what metrics can be used to identify cheaters in a video game. Some domain knowledge of CS:GO is helpful as well as how cheating works. I've already mentioned above why the number of games owned might help build a more effective model. Any metric that is available either directly or through a calculation indicating a players performance should be obvious choices since cheating should allow them to perform way better than the rest of the players.

None of the columns in the dataset contain a direct measure of accuracy, but that's easily solved with a little bit of math. Since we have metrics around the total number of shots overall (total_shots_fired) as well as total shots that hit a player (total_shots_hit) we can very easily come up with an accuracy metric:

``` text
total_accuracy = total_shots_hit / total_shots_fired
```

And then again for each weapon, like so: 

``` text
accuracy_ssg08 = total_hits_ssg08 / total_shots_ssg08
accuracy_awp = total_hits_awp / total_shots_awp
accuracy_deagle = total_hits_deagle / total_shots_deagle
accuracy_aug = total_hits_aug / total_shots_aug
accuracy_scar20 = total_hits_scar20 / total_shots_scar20
accuracy_m4a1 = total_hits_m4a1 / total_shots_m4a1
accuracy_ak47 = total_hits_ak47 / total_shots_ak47
accuracy_bizon = total_hits_bizon / total_shots_bizon
accuracy_elite = total_hits_elite / total_shots_elite
accuracy_famas = total_hits_famas / total_shots_famas
accuracy_fiveseven = total_hits_fiveseven / total_shots_fiveseven
accuracy_g3sg1 = total_hits_g3sg1 / total_shots_g3sg1
accuracy_galilar = total_hits_galilar / total_shots_galilar
accuracy_glock = total_hits_glock / total_shots_glock
accuracy_hkp2000 = total_hits_hkp2000 / total_shots_hkp2000
accuracy_m249 = total_hits_m249 / total_shots_m249
accuracy_mac10 = total_hits_mac10 / total_shots_mac10
accuracy_mag7 = total_hits_mag7 / total_shots_mag7
accuracy_mp7 = total_hits_mp7 / total_shots_mp7
accuracy_mp9 = total_hits_mp9 / total_shots_mp9
accuracy_negev = total_hits_negev / total_shots_negev
accuracy_nova = total_hits_nova / total_shots_nova
accuracy_p250 = total_hits_p250 / total_shots_p250
accuracy_p90 = total_hits_p90 / total_shots_p90
accuracy_sawedoff = total_hits_sawedoff / total_shots_sawedoff
accuracy_sg556 = total_hits_sg556 / total_shots_sg556
accuracy_tec9 = total_hits_tec9 / total_shots_tec9
accuracy_ump45 = total_hits_ump45 / total_shots_ump45
accuracy_xm1014 = total_hits_xm1014 / total_shots_xm1014
```
Finally we can come up with some performance ratio's that create a metrics that ties to how well they are playing and can easily be compared:

``` text
win_ratio = total_wins / total_rounds_played
kill_to_death_ratio = total_kills / total_deaths
total_wins_per_hour = ((total_wins / total_time_played) / 60) / 60 (seconds -> hours)
mvp_per_round = total_mvps / total_rounds_played
total_headshots_per_round = total_kills_headshot / total_rounds_played
```

Another important consideration when choosing what data to include has to do with data quality. Here are some additional rules that were considered when choosing data to build the model:

1. Exclude a Steam ID if the player doesn't own CS:GO or has a Private Steam Profile.
2. Exclude a Steam ID that was VAC Banned before CS:GO was released. Being banned before the game existed means they were banned cheating at another game.
3. Drop mathematically impossible statistics returned by the Steam API for a given Steam ID (range checks, etc. No data set is clean)
4. Drop Players that haven't played enough:
 * Drop rows in which a player has less than 100 frags
 * Drop rows if player has less than four (4) hours of play time.

Having excluded data that's not ideal for building the model and calculating metrics, a CSV file containing these data points would look something like this:

``` csv
SteamID,win_ratio,total_accuracy,kill_to_death_ratio,total_wins_per_hour,mvp_per_round,total_headshots_per_round,accuracy_ssg08,accuracy_awp,accuracy_deagle,accuracy_aug,accuracy_scar20,accuracy_m4a1,accuracy_ak47,accuracy_bizon,accuracy_elite,accuracy_famas,accuracy_fiveseven,accuracy_g3sg1,accuracy_galilar,accuracy_glock,accuracy_hkp2000,accuracy_m249,accuracy_mac10,accuracy_mag7,accuracy_mp7,accuracy_mp9,accuracy_negev,accuracy_nova,accuracy_p250,accuracy_p90,accuracy_sawedoff,accuracy_sg556,accuracy_tec9,accuracy_ump45,accuracy_xm1014,total_games_owned,VACBanned
76561197960266309,0.525736912,0.247670357,1.90780365,21.2591071,0.128171286,0.618419123,0.516431925,0.38559322,0.239507959,0.438261442,0.365429234,0.198184254,0.165430106,0.59509434,0.309309309,0.220901639,0.218390805,0.724747475,0.23771823,0.211877695,0.186304837,0.354302242,0.411804158,0.255145377,0.572246696,0.434482759,0.873367439,0.245160123,0.241709654,0.342210968,0.185152452,0.194079733,0.192628851,0.400611621,0.302351212,43,1
76561197960267921,0.49957584,0.169590373,1.147440713,23.92076508,0.109602986,0.268959959,0.443438914,0.427052857,0.244571999,0.18877551,0.151315789,0.179089842,0.171991996,0.161024486,0.170977011,0.161507402,0.214285714,0.128851541,0.142695357,0.175239214,0.188723835,0.055555556,0.137795276,0.180434783,0.16800428,0.231155779,0.035989717,0.175792507,0.187616624,0.15185354,0.111111111,0.114345114,0.126506024,0.144204852,0.205882353,35,1
76561197960268033,0.688679245,0.130749237,1.44612069,20.08713597,0.400943396,1.636792453,0.376237624,0.272727273,0.220973783,0.147559591,0.352272727,0.13485064,0.130980225,0.146263911,,0.134615385,0.164383562,0.333333333,0.1485623,0.2,0.221105528,0.039571695,0.048387097,0.216248507,0.161676647,0.129032258,0.077586207,0.12962963,0.172839506,,,0.127016129,0.058510638,0.082089552,,19,1
76561197960269346,0.513513514,0.172496372,1.229127433,26.67597192,0.129919393,0.392128971,0.329411765,0.406060606,0.182481752,0.182565789,0.134092901,0.207852564,0.182166215,0.169014085,0.04,0.185106383,0.261904762,0.113716295,0.187698161,0.155621742,0.188359788,0.054794521,0.117647059,0.238363893,0.277777778,0.138686131,0.029676259,0.172979798,0.227765727,0.158285243,0.131868132,0.151335312,0.163636364,0.160933661,,32,1
76561197960269354,0.43373494,0.157913165,1.019047619,9.9493321,0.036144578,1.21686747,0.4,0.34,0.205426357,0.279661017,0.133333333,0.173986486,0.139305446,0.166666667,0.214285714,0.071428571,0.6,,0.363636364,0.103448276,0.109947644,,0.125,0.170940171,0.204899777,0.194528875,,0.111111111,0.178571429,0.137566138,0.118518519,0.102272727,0.325581395,0.14516129,0.151515152,25,1
76561197960269370,0.513513514,0.17114094,0.829896907,15.42857147,0.099099099,0.585585586,0.25,0.34,0.2,0.173076923,0.363636364,0.180161943,0.174852652,0.140625,0.095890411,0.188172043,,0.285714286,0.143617021,0.14516129,0.163934426,0.021276596,0.168421053,0.248366013,0.148535565,0.1,,0.164750958,0.128205128,0.196319018,0.141975309,0.096153846,0.204819277,0.076923077,0.216216216,70,1
76561197960269451,0.470016207,0.181190032,0.27057903,16.38786711,0.106428957,0.437601297,0.24,0.340172786,0.225092251,0.047619048,0.098360656,0.165487769,0.155614209,0.152019002,1.05,0.146959459,0.183544304,0.233333333,0.168539326,0.20302088,0.210718636,,,0.181978799,0.168141593,,0.117647059,0.240079365,0.375496689,0.163116428,0.162393162,0.25,,0.097959184,0.231481481,13,1
76561197960269848,0.536148934,0.189057591,1.363779528,21.54930259,0.058561272,0.751656826,0.549019608,0.5201123,0.306022409,0.023255814,0.131753555,0.172231637,0.169304034,0.136038186,0.132231405,0.183089431,0.244668246,0.075144509,0.142957995,0.176989557,0.203314138,0.019933555,0.047318612,0.13036566,0.10428737,0.118811881,0.03657418,0.169082126,0.196078431,0.089637635,0.14379085,0.088757396,0.206896552,0.122137405,0.176470588,22,1
76561197960270064,0.555710306,0.161824082,1.457808564,23.47288949,0.16643454,0.701949861,0.197674419,0.301847437,0.17879787,0.228395062,0.088235294,0.159228527,0.129699406,0.194312796,0.166986564,0.142424242,0.118055556,0.19047619,0.115942029,0.167520117,0.16023166,0.102040816,0.16,0.159883721,0.206081081,0.143678161,0.065656566,0.17988008,0.154444444,0.155789474,0.159090909,0.219298246,0.195121951,0.203463203,0.204255319,76,1
...etc...

```

Notice that some of the columns are empty - that's okay, the Nexosis API has data imputation strategies internally that help solve for that.

The sample dataset originally used in building this model had around 21,000 CSGO player metrics and was aprox. 9MB on disk.

## Uploading the Data

For the simplification of this model and sample so it doesn't use up the Nexosis Quotas on our Community Pricing Tier, we've prepared a smaller sub-set of the data which scores about over 5% less accurate but is good enough to follow along and understand what's happening and get some decent results.

> When you create an account with Nexosis and chose to include the sample datasets in your account, this dataset will be pre-loaded and is named 'CSGO-Stats'. We've also included it in our `sampledata` github repo here at [CSGO DataSet on Git](https://github.com/Nexosis/sampledata/blob/master/csgo-small.csv){:target="_blank"}

If the SteamID is included when you upload the dataset, the API needs to know to treat it as a primary identifier (much like a database primary key) which requires the Columns metadata set to indicate it's Role.  To simplify, we can exclude this column as well since we don't plan on matching the model results back up with the original dataset and build the model without it.

To import from our GitHub URL linked above `POST` to `https://ml.nexosis.com/v1/imports/Url` and set the BODY to `{"contentType": "csv","url":"https://raw.githubusercontent.com/Nexosis/sampledata/master/csgo-small.csv","dataSetName":"CSGO-Stats"}` like so:

<ul id="profileTabs" class="nav nav-tabs">
    <li class="active"><a href="#curlUploadData" data-toggle="tab">Curl</a></li>
    <li><a href="#powershellUploadData" data-toggle="tab">Powershell</a></li>
</ul>
<div class="tab-content">
    <div role="tabpanel" class="tab-pane active" id="curlUploadData">
    <p>The following command assumes your Nexosis API Key is stored in an Environment Variable called <code>$NEXOSIS_API_KEY</code>.</p><pre class="language-bash"><code class="language-bash code-toolbar">$ curl -v -X POST "https://ml.nexosis.com/v1/imports/Url" -s \
        -H "Content-Type: application/json" \
        -H "api-key: $NEXOSIS_API_KEY" \
        -d '{"contentType": "csv","url":"https://raw.githubusercontent.com/Nexosis/sampledata/master/csgo-small.csv","dataSetName":"CSGO-Stats"}'</code></pre>Formatted HTTP Response:<pre class="language-json" style="max-height:30em;"><code class="language-json code-toolbar">{
    "importId": "0160dce4-1f9a-42c6-a0ab-b5c2bcd55415",
    "type": "url",
    "status": "requested",
    "dataSetName": "CSGO-Stats",
    "parameters": {
        "url": "https://raw.githubusercontent.com/Nexosis/sampledata/master/csgo-small.csv"
    },
    "requestedDate": "2018-01-09T21:47:06.010512+00:00",
    "statusHistory": [{
        "date": "2018-01-09T21:47:06.010512+00:00",
        "status": "requested"
    }],
    "messages": [],
    "columns": {},
    "links": [{
        "rel": "self",
        "href": "https://ml.nexosis.com/v1/imports/Url"
    }, {
        "rel": "data",
        "href": "https://ml.nexosis.com/v1/data/CSGO-Stats"
    }]
}</code></pre>
    </div>
    <div role="tabpanel" class="tab-pane" id="powershellUploadData">
      <p>The following command assumes your Steam API Key is stored in an Environment Variable called <code>$NEXOSIS_API_KEY</code>.</p>
      <pre class="language-powershell"><code class="language-powershell code-toolbar">PS> Import-NexosisDataSetFromUrl -url https://raw.githubusercontent.com/Nexosis/sampledata/master/csgo-small.csv -dataSetName CSGO-Stats -contentType csv

importId      : 0160dcdb-48ab-43e9-8465-10c9f0047183
type          : url
status        : requested
dataSetName   : CSGO-Stats
parameters    : @{url=https://raw.githubusercontent.com/Nexosis/sampledata/master/csgo-small.csv}
requestedDate : 2018-01-09T21:37:26.678844+00:00
statusHistory : {@{date=2018-01-09T21:37:26.678844+00:00; status=requested}}
messages      : {}
columns       : 
links         : {@{rel=self; href=https://ml.nexosis.com/v1/imports/Url}, @{rel=data; href=https://ml.nexosis.com/v1/data/CSGO-Stats}}
  </code>
</pre>
    </div>
</div>

Once the import has been queued, we can check on the `status` to be `completed`:

<ul id="profileTabs" class="nav nav-tabs">
    <li class="active"><a href="#curlUploadDataStatus" data-toggle="tab">Curl</a></li>
    <li><a href="#powershellUploadDataStatus" data-toggle="tab">Powershell</a></li>
</ul>
<div class="tab-content">
    <div role="tabpanel" class="tab-pane active" id="curlUploadDataStatus">
    <p>The following command assumes your Nexosis API Key is stored in an Environment Variable called <code>$NEXOSIS_API_KEY</code>.</p><pre class="language-bash"><code class="language-bash code-toolbar">$ curl -X GET "https://ml.nexosis.com/v1/imports/0160dcdb-48ab-43e9-8465-10c9f0047183" -s \
                -H "api-key: $NEXOSIS_API_KEY"</code></pre>Formatted HTTP Response:<pre class="language-json" style="max-height:30em;"><code class="language-json code-toolbar">{
    "importId": "0160dcdb-48ab-43e9-8465-10c9f0047183",
    "type": "url",
    "status": "completed",
    "dataSetName": "CSGO-Stats",
    "parameters": {
        "url": "https://raw.githubusercontent.com/Nexosis/sampledata/master/csgo-small.csv"
    },
    "requestedDate": "2018-01-09T21:37:26.678844+00:00",
    "statusHistory": [{
        "date": "2018-01-09T21:37:26.678844+00:00",
        "status": "requested"
    }, {
        "date": "2018-01-09T21:37:28.2196397+00:00",
        "status": "started"
    }, {
        "date": "2018-01-09T21:37:30.8427806+00:00",
        "status": "completed"
    }],
    "messages": [],
    "columns": {},
    "links": [{
        "rel": "self",
        "href": "https://ml.nexosis.com/v1/imports/0160dcdb-48ab-43e9-8465-10c9f0047183"
    }, {
        "rel": "data",
        "href": "https://ml.nexosis.com/v1/data/CSGO-Stats"
    }]
}</code></pre>
    </div>
    <div role="tabpanel" class="tab-pane" id="powershellUploadDataStatus">
      <p>The following command assumes your Steam API Key is stored in an Environment Variable called <code>$NEXOSIS_API_KEY</code>.</p>
      <pre class="language-powershell"><code class="language-powershell code-toolbar">PS> Get-NexosisImport -importId 0160dcdb-48ab-43e9-8465-10c9f0047183

importId      : 0160dcdb-48ab-43e9-8465-10c9f0047183
type          : url
status        : completed
dataSetName   : CSGO-Stats
parameters    : @{url=https://raw.githubusercontent.com/Nexosis/sampledata/master/csgo-small.csv}
requestedDate : 2018-01-09T21:37:26.678844+00:00
statusHistory : {@{date=2018-01-09T21:37:26.678844+00:00; status=requested}, @{date=2018-01-09T21:37:28.2196397+00:00; status=started}, @{date=2018-01-09T21:37:30.8427806+00:00; 
                status=completed}}
messages      : {}
columns       : 
links         : {@{rel=self; href=https://ml.nexosis.com/v1/imports/0160dcdb-48ab-43e9-8465-10c9f0047183}, @{rel=data; href=https://ml.nexosis.com/v1/data/CSGO-Stats}}
  </code>
</pre>
    </div>
</div>

## Creating a Classification Model

Now the the data is uploaded, a model can be built using CSGO player statistics to predict if a player has potentially been cheating:

To build a classification model, `POST` to `https://ml.nexosis.com/v1/sessions/model` to start a Model building session:

<ul id="profileTabs" class="nav nav-tabs">
    <li class="active"><a href="#curlBuildModel" data-toggle="tab">Curl</a></li>
    <li><a href="#powershellBuildModel" data-toggle="tab">Powershell</a></li>
</ul>
<div class="tab-content">
    <div role="tabpanel" class="tab-pane active" id="curlBuildModel">
    <p>The following command assumes your Nexosis API Key is stored in an Environment Variable called <code>$NEXOSIS_API_KEY</code>.</p><pre class="language-bash"><code class="language-bash code-toolbar">$ curl -s -X POST "https://ml.nexosis.com/v1/sessions/model" \
        -H "Content-Type: application/json" \
        -H "api-key: $NEXOSIS_API_KEY" \
        -d '{"dataSourceName":"csgo-stats","predictionDomain":"classification","targetColumn": "VACBanned"}'</code></pre>Formatted HTTP Response:<pre class="language-json" style="max-height:30em;"><code class="language-json code-toolbar">{
    "columns": {
        "VACBanned": {
            "dataType": "numeric",
            "role": "target",
            "imputation": "mode",
            "aggregation": "mode"
        },
        "win_ratio": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_aug": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_awp": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_mp7": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_mp9": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_p90": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_ak47": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_m249": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_m4a1": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_mag7": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_nova": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_p250": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_tec9": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "mvp_per_round": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_bizon": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_elite": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_famas": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_g3sg1": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_glock": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_mac10": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_negev": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_sg556": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_ssg08": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_ump45": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "total_accuracy": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_deagle": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_scar20": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_xm1014": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_galilar": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_hkp2000": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_sawedoff": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "total_games_owned": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_fiveseven": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "kill_to_death_ratio": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "total_wins_per_hour": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "total_headshots_per_round": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        }
    },
    "sessionId": "0160dcf9-09a9-4c1a-9f5a-75f066ee886c",
    "type": "model",
    "status": "requested",
    "predictionDomain": "classification",
    "availablePredictionIntervals": [],
    "requestedDate": "2018-01-09T22:09:56.647007+00:00",
    "statusHistory": [{
        "date": "2018-01-09T22:09:56.647007+00:00",
        "status": "requested"
    }],
    "extraParameters": {
        "balance": true
    },
    "messages": [],
    "dataSourceName": "csgo-stats",
    "dataSetName": "csgo-stats",
    "targetColumn": "VACBanned",
    "isEstimate": false,
    "links": [{
        "rel": "results",
        "href": "https://ml.nexosis.com/v1/sessions/0160dcf9-09a9-4c1a-9f5a-75f066ee886c/results"
    }, {
        "rel": "data",
        "href": "https://ml.nexosis.com/v1/data/csgo-stats"
    }]
}</code></pre>
    </div>
    <div role="tabpanel" class="tab-pane" id="powershellBuildModel">
      <p>The following command assumes your Steam API Key is stored in an Environment Variable called <code>$NEXOSIS_API_KEY</code>.</p>
      <pre class="language-powershell"><code class="language-powershell code-toolbar">PS> Start-NexosisModelSession -dataSourceName csgo -targetColumn VACBanned -predictionDomain Classification

columns                      : @{win_ratio=; accuracy_aug=; accuracy_awp=; accuracy_mp7=; accuracy_mp9=; accuracy_p90=; accuracy_ak47=; accuracy_m249=; accuracy_m4a1=; 
                               accuracy_mag7=; accuracy_nova=; accuracy_p250=; accuracy_tec9=; mvp_per_round=; accuracy_bizon=; accuracy_elite=; accuracy_famas=; accuracy_g3sg1=; 
                               accuracy_glock=; accuracy_mac10=; accuracy_negev=; accuracy_sg556=; accuracy_ssg08=; accuracy_ump45=; total_accuracy=; accuracy_deagle=; accuracy_scar20=; 
                               accuracy_xm1014=; accuracy_galilar=; accuracy_hkp2000=; accuracy_sawedoff=; total_games_owned=; accuracy_fiveseven=; kill_to_death_ratio=; 
                               total_wins_per_hour=; total_headshots_per_round=}
sessionId                    : 016050d8-8b42-44bc-9c2b-360a240d6aec
type                         : model
status                       : requested
predictionDomain             : classification
availablePredictionIntervals : {}
requestedDate                : 2017-12-13T17:07:36.862739+00:00
statusHistory                : {@{date=2017-12-13T17:07:36.862739+00:00; status=requested}}
extraParameters              : @{balance=True}
messages                     : {}
dataSourceName               : csgo
dataSetName                  : csgo
targetColumn                 : VACBanned
links                        : {@{rel=results; href=https://ml.nexosisdev.com/v1/sessions/016050d8-8b42-44bc-9c2b-360a240d6aec/results}, @{rel=data; 
                               href=https://ml.nexosisdev.com/v1/data/csgo}}
costEstimate                 : 0.00 USD
  </code>
</pre>
    </div>
</div>

Since building a model is computationally expensive, it's not instantaneous - we have to wait until it's complete before we can use it. To check to see if the model is complete, submit a `HEAD` request to `https://ml.nexosis.com/v1/sessions/{sessionId}` using the session ID returned from the previous step:

<ul id="profileTabs" class="nav nav-tabs">
    <li class="active"><a href="#curlModelStatus" data-toggle="tab">Curl</a></li>
    <li><a href="#powershellModelStatus" data-toggle="tab">Powershell</a></li>
</ul>
<div class="tab-content">
    <div role="tabpanel" class="tab-pane active" id="curlModelStatus">
    <p>The following command assumes your Nexosis API Key is stored in an Environment Variable called <code>$NEXOSIS_API_KEY</code>.</p><pre class="language-bash"><code class="language-bash code-toolbar">$ curl -X HEAD "https://ml.nexosis.com/v1/sessions/016050d8-8b42-44bc-9c2b-360a240d6aec" \
     -H "api-key: $NEXOSIS_API_KEY" --head -s</code></pre>HTTP Headers:<pre class="language-test" style="max-height:30em;"><code class="language-test code-toolbar">HTTP/1.1 200 OK
Content-Length: 0
Nexosis-Session-Status: Completed
Nexosis-Account-DataSetCount-Allotted: 200
Nexosis-Account-DataSetCount-Current: 199
Nexosis-Account-PredictionCount-Allotted: 250000
Nexosis-Account-PredictionCount-Current: 0
Nexosis-Account-SessionCount-Allotted: 3500
Nexosis-Account-SessionCount-Current: 0
Nexosis-Account-Balance: 0.00 USD
Nexosis-Request-Cost: 0.00 USD
Date: Tue, 09 Jan 2018 23:27:18 GMT</code></pre>
    </div>
    <div role="tabpanel" class="tab-pane" id="powershellModelStatus">
      <p>The following command assumes your Steam API Key is stored in an Environment Variable called <code>$NEXOSIS_API_KEY</code>.</p>
      <pre class="language-powershell"><code class="language-powershell code-toolbar">PS> Get-NexosisSessionStatus -SessionId 016050d8-8b42-44bc-9c2b-360a240d6aec

Completed</code>
</pre>
    </div>
</div>

Once the `Nexosis-Session-Status` header returns `Completed` the session results can be retrieved which will contain a Model ID, winning algorithm, and accuracy metrics.

## Reviewing the Session Results and Model Accuracy

Once the Model Building Session is complete you can retrive the results issuing a `GET` request to `https://ml.nexosis.com/v1/sessions/{modelid}/results` like so:

<ul id="profileTabs" class="nav nav-tabs">
    <li class="active"><a href="#curlSessionResults" data-toggle="tab">Curl</a></li>
    <li><a href="#powershellSessionResults" data-toggle="tab">Powershell</a></li>
</ul>
<div class="tab-content">
    <div role="tabpanel" class="tab-pane active" id="curlSessionResults">
    <p>The following command assumes your Nexosis API Key is stored in an Environment Variable called <code>$NEXOSIS_API_KEY</code>.</p><pre class="language-bash"><code class="language-bash code-toolbar">$ curl -X GET "https://ml.nexosis.com/v1/sessions/016050d8-8b42-44bc-9c2b-360a240d6aec/results"\
    -H "api-key: $NEXOSIS_API_KEY" -s</code></pre>Formatted HTTP Response:<pre class="language-json" style="max-height:30em;"><code class="language-json code-toolbar">{
    "metrics": {
        "macroAverageF1Score": 0.72521972411860092,
        "rocAreaUnderCurve": 0.79444811632518164,
        "accuracy": 0.72888888888888892,
        "macroAveragePrecision": 0.72667984189723323,
        "macroAverageRecall": 0.73690515532055523,
        "matthewsCorrelationCoefficient": 0.46347221341824979
    },
    "data": [{
        "VACBanned": "0",
        "win_ratio": "0.51",
        "accuracy_aug": "0.22",
        "accuracy_awp": "0.4",
        "accuracy_mp7": "0.18",
        "accuracy_mp9": "0.17",
        "accuracy_p90": "0.15",
        "accuracy_ak47": "0.2",
        "accuracy_m249": "0.1",
        "accuracy_m4a1": "0.22",
        "accuracy_mag7": "0.23",
        "accuracy_nova": "0.19",
        "accuracy_p250": "0.22",
        "accuracy_tec9": "0.17",
        "mvp_per_round": "0.09",
        "accuracy_bizon": "0.14",
        "accuracy_elite": "0.18",
        "accuracy_famas": "0.2",
        "accuracy_g3sg1": "0.14",
        "accuracy_glock": "0.18",
        "accuracy_mac10": "0.18",
        "accuracy_negev": "0.05",
        "accuracy_sg556": "0.19",
        "accuracy_ssg08": "0.39",
        "accuracy_ump45": "0.18",
        "total_accuracy": "0.2",
        "accuracy_deagle": "0.26",
        "accuracy_scar20": "0.18",
        "accuracy_xm1014": "0.18",
        "VACBanned:actual": "0",
        "accuracy_galilar": "0.15",
        "accuracy_hkp2000": "0.19",
        "accuracy_sawedoff": "0.16",
        "total_games_owned": "51",
        "accuracy_fiveseven": "0.22",
        "kill_to_death_ratio": "1.3",
        "total_wins_per_hour": "15.13",
        "total_headshots_per_round": "1.06"
    }, {
        "VACBanned": "0",
        "win_ratio": "0.51",
        "accuracy_aug": "0.24",
        "accuracy_awp": "0.36",
        "accuracy_mp7": "0.17",
        "accuracy_mp9": "0.2",
        "accuracy_p90": "0.15",
        "accuracy_ak47": "0.2",
        "accuracy_m249": "0.1",
        "accuracy_m4a1": "0.19",
        "accuracy_mag7": "0.23",
        "accuracy_nova": "0.09",
        "accuracy_p250": "0.21",
        "accuracy_tec9": "0.06",
        "mvp_per_round": "0.1",
        "accuracy_bizon": "0.17",
        "accuracy_elite": "0.14",
        "accuracy_famas": "0.23",
        "accuracy_g3sg1": "0.23",
        "accuracy_glock": "0.16",
        "accuracy_mac10": "0.13",
        "accuracy_negev": "0.06",
        "accuracy_sg556": "0.19",
        "accuracy_ssg08": "0.24",
        "accuracy_ump45": "0.18",
        "total_accuracy": "0.19",
        "accuracy_deagle": "0.27",
        "accuracy_scar20": "0.14",
        "accuracy_xm1014": "0.18",
        "VACBanned:actual": "0",
        "accuracy_galilar": "0.19",
        "accuracy_hkp2000": "0.17",
        "accuracy_sawedoff": "0.21",
        "total_games_owned": "42",
        "accuracy_fiveseven": "0.18",
        "kill_to_death_ratio": "1.01",
        "total_wins_per_hour": "20.17",
        "total_headshots_per_round": "0.63"
    }, {
        "VACBanned": "0",
        "win_ratio": "0.5",
        "accuracy_aug": "0.17",
        "accuracy_awp": "0.35",
        "accuracy_mp7": "0.2",
        "accuracy_mp9": "0.09",
        "accuracy_p90": "0.13",
        "accuracy_ak47": "0.16",
        "accuracy_m249": "0.09",
        "accuracy_m4a1": "0.2",
        "accuracy_mag7": "0.26",
        "accuracy_nova": "0.17",
        "accuracy_p250": "0.17",
        "accuracy_tec9": "",
        "mvp_per_round": "0.12",
        "accuracy_bizon": "",
        "accuracy_elite": "",
        "accuracy_famas": "0.2",
        "accuracy_g3sg1": "0.15",
        "accuracy_glock": "0.17",
        "accuracy_mac10": "0.04",
        "accuracy_negev": "0.18",
        "accuracy_sg556": "0.15",
        "accuracy_ssg08": "0.24",
        "accuracy_ump45": "0.23",
        "total_accuracy": "0.18",
        "accuracy_deagle": "0.19",
        "accuracy_scar20": "0.17",
        "accuracy_xm1014": "",
        "VACBanned:actual": "0",
        "accuracy_galilar": "0.14",
        "accuracy_hkp2000": "0.19",
        "accuracy_sawedoff": "",
        "total_games_owned": "2",
        "accuracy_fiveseven": "0.18",
        "kill_to_death_ratio": "1.21",
        "total_wins_per_hour": "24.11",
        "total_headshots_per_round": "0.31"
    },
    {
            // removed many many rows of data
    }],
    "pageNumber": 0,
    "totalPages": 5,
    "pageSize": 50,
    "totalCount": 225,
    "sessionId": "016050d8-8b42-44bc-9c2b-360a240d6aec",
    "type": "model",
    "status": "completed",
    "predictionDomain": "classification",
    "availablePredictionIntervals": [],
    "modelId": "1b79d672-99a4-47f7-a387-a001d9420220",
    "requestedDate": "2018-01-10T02:03:11.832359+00:00",
    "statusHistory": [{
        "date": "2018-01-10T02:03:11.832359+00:00",
        "status": "requested"
    }, {
        "date": "2018-01-10T02:03:13.0436711+00:00",
        "status": "started"
    }, {
        "date": "2018-01-10T02:06:17.7404773+00:00",
        "status": "completed"
    }],
    "extraParameters": {
        "balance": true
    },
    "messages": [{
        "severity": "informational",
        "message": "1126 observations were found in the dataset."
    }],
    "dataSourceName": "csgo-stats",
    "dataSetName": "csgo-stats",
    "targetColumn": "VACBanned",
    "links": [{
        "rel": "self",
        "href": "https://ml.nexosis.com/v1/sessions/016050d8-8b42-44bc-9c2b-360a240d6aec/results"
    }, {
        "rel": "model",
        "href": "https://ml.nexosis.com/v1/models/1b79d672-99a4-47f7-a387-a001d9420220"
    }, {
        "rel": "confusionMatrix",
        "href": "https://ml.nexosis.com/v1/sessions/016050d8-8b42-44bc-9c2b-360a240d6aec/results/confusionmatrix"
    }, {
        "rel": "classScores",
        "href": "https://ml.nexosis.com/v1/sessions/016050d8-8b42-44bc-9c2b-360a240d6aec/results/classScores"
    }, {
        "rel": "contest",
        "href": "https://ml.nexosis.com/v1/sessions/016050d8-8b42-44bc-9c2b-360a240d6aec/contest"
    }, {
        "rel": "first",
        "href": "https://ml.nexosis.com/v1/sessions/016050d8-8b42-44bc-9c2b-360a240d6aec/results?page=0"
    }, {
        "rel": "next",
        "href": "https://ml.nexosis.com/v1/sessions/016050d8-8b42-44bc-9c2b-360a240d6aec/results?page=1"
    }, {
        "rel": "last",
        "href": "https://ml.nexosis.com/v1/sessions/016050d8-8b42-44bc-9c2b-360a240d6aec/results?page=4"
    }]
}</code></pre>
    </div>
    <div role="tabpanel" class="tab-pane" id="powershellSessionResults">
      <p>The following command assumes your Steam API Key is stored in an Environment Variable called <code>$NEXOSIS_API_KEY</code>.</p>
      <pre class="language-powershell"><code class="language-powershell code-toolbar">PS> Get-NexosisSessionResult -SessionId 016050d8-8b42-44bc-9c2b-360a240d6aec</code></pre>Output:<pre class="language-json" style="max-height:30em;"><code class="language-json code-toolbar">metrics                      : @{macroAverageF1Score=0.72521972411860092; rocAreaUnderCurve=0.79444811632518164; 
                               accuracy=0.72888888888888892; macroAveragePrecision=0.72667984189723323; 
                               macroAverageRecall=0.73690515532055523; matthewsCorrelationCoefficient=0.46347221341824979}
data                         : {@{VACBanned=0; win_ratio=0.51; accuracy_aug=0.22; accuracy_awp=0.4; accuracy_mp7=0.18; accuracy_mp9=0.17; 
                               accuracy_p90=0.15; accuracy_ak47=0.2; accuracy_m249=0.1; accuracy_m4a1=0.22; accuracy_mag7=0.23; 
                               accuracy_nova=0.19; accuracy_p250=0.22; accuracy_tec9=0.17; mvp_per_round=0.09; accuracy_bizon=0.14; 
                               accuracy_elite=0.18; accuracy_famas=0.2; accuracy_g3sg1=0.14; accuracy_glock=0.18; accuracy_mac10=0.18; 
                               accuracy_negev=0.05; accuracy_sg556=0.19; accuracy_ssg08=0.39; accuracy_ump45=0.18; total_accuracy=0.2; 
                               accuracy_deagle=0.26; accuracy_scar20=0.18; accuracy_xm1014=0.18; VACBanned:actual=0; accuracy_galilar=0.15; 
                               accuracy_hkp2000=0.19; accuracy_sawedoff=0.16; total_games_owned=51; accuracy_fiveseven=0.22; 
                               kill_to_death_ratio=1.3; total_wins_per_hour=15.13; total_headshots_per_round=1.06}, @{VACBanned=0; 
                               win_ratio=0.51; accuracy_aug=0.24; accuracy_awp=0.36; accuracy_mp7=0.17; accuracy_mp9=0.2; accuracy_p90=0.15; 
                               accuracy_ak47=0.2; accuracy_m249=0.1; accuracy_m4a1=0.19; accuracy_mag7=0.23; accuracy_nova=0.09; 
                               accuracy_p250=0.21; accuracy_tec9=0.06; mvp_per_round=0.1; accuracy_bizon=0.17; accuracy_elite=0.14; 
                               accuracy_famas=0.23; accuracy_g3sg1=0.23; accuracy_glock=0.16; accuracy_mac10=0.13; accuracy_negev=0.06; 
                               accuracy_sg556=0.19; accuracy_ssg08=0.24; accuracy_ump45=0.18; total_accuracy=0.19; accuracy_deagle=0.27; 
                               accuracy_scar20=0.14; accuracy_xm1014=0.18; VACBanned:actual=0; accuracy_galilar=0.19; accuracy_hkp2000=0.17; 
                               accuracy_sawedoff=0.21; total_games_owned=42; accuracy_fiveseven=0.18; kill_to_death_ratio=1.01; 
                               total_wins_per_hour=20.17; total_headshots_per_round=0.63}, @{VACBanned=0; win_ratio=0.5; accuracy_aug=0.17; 
                               accuracy_awp=0.35; accuracy_mp7=0.2; accuracy_mp9=0.09; accuracy_p90=0.13; accuracy_ak47=0.16; 
                               accuracy_m249=0.09; accuracy_m4a1=0.2; accuracy_mag7=0.26; accuracy_nova=0.17; accuracy_p250=0.17; 
                               accuracy_tec9=; mvp_per_round=0.12; accuracy_bizon=; accuracy_elite=; accuracy_famas=0.2; 
                               accuracy_g3sg1=0.15; accuracy_glock=0.17; accuracy_mac10=0.04; accuracy_negev=0.18; accuracy_sg556=0.15; 
                               accuracy_ssg08=0.24; accuracy_ump45=0.23; total_accuracy=0.18; accuracy_deagle=0.19; accuracy_scar20=0.17; 
                               accuracy_xm1014=; VACBanned:actual=0; accuracy_galilar=0.14; accuracy_hkp2000=0.19; accuracy_sawedoff=; 
                               total_games_owned=2; accuracy_fiveseven=0.18; kill_to_death_ratio=1.21; total_wins_per_hour=24.11; 
                               total_headshots_per_round=0.31}, @{VACBanned=0; win_ratio=0.49; accuracy_aug=0.21; accuracy_awp=0.3; 
                               accuracy_mp7=0.26; accuracy_mp9=0.19; accuracy_p90=0.14; accuracy_ak47=0.16; accuracy_m249=0.13; 
                               accuracy_m4a1=0.19; accuracy_mag7=0.15; accuracy_nova=0.17; accuracy_p250=0.19; accuracy_tec9=0.17; 
                               mvp_per_round=0.1; accuracy_bizon=0.18; accuracy_elite=0.24; accuracy_famas=0.2; accuracy_g3sg1=0.11; 
                               accuracy_glock=0.17; accuracy_mac10=0.19; accuracy_negev=0.08; accuracy_sg556=0.17; accuracy_ssg08=0.38; 
                               accuracy_ump45=0.18; total_accuracy=0.15; accuracy_deagle=0.23; accuracy_scar20=0.15; accuracy_xm1014=0.15; 
                               VACBanned:actual=0; accuracy_galilar=0.15; accuracy_hkp2000=0.18; accuracy_sawedoff=0.15; 
                               total_games_owned=23; accuracy_fiveseven=0.19; kill_to_death_ratio=1.13; total_wins_per_hour=22.92; 
                               total_headshots_per_round=0.42}...}
pageNumber                   : 0
totalPages                   : 5
pageSize                     : 50
totalCount                   : 225
sessionId                    : 0160ddce-965a-4281-8fd7-6a58e1cbba4b
type                         : model
status                       : completed
predictionDomain             : classification
availablePredictionIntervals : {}
modelId                      : 1b79d672-99a4-47f7-a387-a001d9420220
requestedDate                : 2018-01-10T02:03:11.832359+00:00
statusHistory                : {@{date=2018-01-10T02:03:11.832359+00:00; status=requested}, @{date=2018-01-10T02:03:13.0436711+00:00; 
                               status=started}, @{date=2018-01-10T02:06:17.7404773+00:00; status=completed}}
extraParameters              : @{balance=True}
messages                     : {@{severity=informational; message=1126 observations were found in the dataset.}}
dataSourceName               : csgo-stats
dataSetName                  : csgo-stats
targetColumn                 : VACBanned
links                        : {@{rel=self; href=https://ml.nexosis.com/v1/sessions/0160ddce-965a-4281-8fd7-6a58e1cbba4b/results}, 
                               @{rel=model; href=https://ml.nexosis.com/v1/models/1b79d672-99a4-47f7-a387-a001d9420220}, 
                               @{rel=confusionMatrix; 
                               href=https://ml.nexosis.com/v1/sessions/0160ddce-965a-4281-8fd7-6a58e1cbba4b/results/confusionmatrix}, 
                               @{rel=classScores; 
                               href=https://ml.nexosis.com/v1/sessions/0160ddce-965a-4281-8fd7-6a58e1cbba4b/results/classScores}...}</code>
</pre>
    </div>
</div>

Once the model has been created, it can be inspected by sending a `GET` to `https://ml.nexosis.com/v1/models/{modelId}`: 

<ul id="profileTabs" class="nav nav-tabs">
    <li class="active"><a href="#curlModel" data-toggle="tab">Curl</a></li>
    <li><a href="#powershellModel" data-toggle="tab">Powershell</a></li>
</ul>
<div class="tab-content">
    <div role="tabpanel" class="tab-pane active" id="curlModel">
    <p>The following command assumes your Nexosis API Key is stored in an Environment Variable called <code>$NEXOSIS_API_KEY</code>.</p><pre class="language-bash"><code class="language-bash code-toolbar">$ curl -X GET "https://ml.nexosis.com/v1/models/1b79d672-99a4-47f7-a387-a001d9420220"\
        -H "api-key: $NEXOSIS_API_KEY" -s</code></pre>Formatted HTTP Response:<pre class="language-json" style="max-height:30em;"><code class="language-json code-toolbar">{
    "columns": {
        "VACBanned": {
            "dataType": "numeric",
            "role": "target",
            "imputation": "mode",
            "aggregation": "mode"
        },
        "win_ratio": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_aug": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_awp": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_mp7": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_mp9": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_p90": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_ak47": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_m249": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_m4a1": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_mag7": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_nova": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_p250": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_tec9": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "mvp_per_round": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_bizon": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_elite": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_famas": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_g3sg1": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_glock": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_mac10": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_negev": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_sg556": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_ssg08": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_ump45": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "total_accuracy": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_deagle": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_scar20": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_xm1014": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_galilar": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_hkp2000": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_sawedoff": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "total_games_owned": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "accuracy_fiveseven": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "kill_to_death_ratio": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "total_wins_per_hour": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        },
        "total_headshots_per_round": {
            "dataType": "numeric",
            "role": "feature",
            "imputation": "zeroes",
            "aggregation": "sum"
        }
    },
    "modelId": "1b79d672-99a4-47f7-a387-a001d9420220",
    "sessionId": "0160ddce-965a-4281-8fd7-6a58e1cbba4b",
    "predictionDomain": "classification",
    "dataSourceName": "csgo-stats",
    "createdDate": "2018-01-10T02:06:06.6928748+00:00",
    "algorithm": {
        "name": "SVC RBF",
        "description": "Support Vector Classification using Radial Basis Function (Gaussian) kernel",
        "key": "svc_rbf"
    },
    "metrics": {
        "macroAverageF1Score": 0.72521972411860092,
        "rocAreaUnderCurve": 0.79444811632518164,
        "accuracy": 0.72888888888888892,
        "macroAveragePrecision": 0.72667984189723323,
        "macroAverageRecall": 0.73690515532055523,
        "matthewsCorrelationCoefficient": 0.46347221341824979
    },
    "links": [{
        "rel": "self",
        "href": "https://ml.nexosis.com/v1/models/1b79d672-99a4-47f7-a387-a001d9420220"
    }, {
        "rel": "train",
        "href": "https://ml.nexosis.com/v1/sessions/0160ddce-965a-4281-8fd7-6a58e1cbba4b"
    }]
}</code></pre>
   </div>
   <div role="tabpanel" class="tab-pane" id="powershellModel">
      <p>The following command assumes your Steam API Key is stored in an Environment Variable called <code>$NEXOSIS_API_KEY</code>.</p>
      <pre class="language-powershell"><code class="language-powershell code-toolbar">PS> Get-NexosisModelDetail -ModelId 1b79d672-99a4-47f7-a387-a001d9420220</code></pre>Output:<pre class="language-powershell" style="max-height:30em;"><code class="language-powershell code-toolbar">columns          : @{VACBanned=; win_ratio=; accuracy_aug=; accuracy_awp=; accuracy_mp7=; accuracy_mp9=; accuracy_p90=; accuracy_ak47=; 
                   accuracy_m249=; accuracy_m4a1=; accuracy_mag7=; accuracy_nova=; accuracy_p250=; accuracy_tec9=; mvp_per_round=; 
                   accuracy_bizon=; accuracy_elite=; accuracy_famas=; accuracy_g3sg1=; accuracy_glock=; accuracy_mac10=; accuracy_negev=; 
                   accuracy_sg556=; accuracy_ssg08=; accuracy_ump45=; total_accuracy=; accuracy_deagle=; accuracy_scar20=; accuracy_xm1014=; 
                   accuracy_galilar=; accuracy_hkp2000=; accuracy_sawedoff=; total_games_owned=; accuracy_fiveseven=; kill_to_death_ratio=; 
                   total_wins_per_hour=; total_headshots_per_round=}
modelId          : 1b79d672-99a4-47f7-a387-a001d9420220
sessionId        : 0160ddce-965a-4281-8fd7-6a58e1cbba4b
predictionDomain : classification
dataSourceName   : csgo-stats
createdDate      : 2018-01-10T02:06:06.6928748+00:00
algorithm        : @{name=SVC RBF; description=Support Vector Classification using Radial Basis Function (Gaussian) kernel; key=svc_rbf}
metrics          : @{macroAverageF1Score=0.72521972411860092; rocAreaUnderCurve=0.79444811632518164; accuracy=0.72888888888888892; 
                   macroAveragePrecision=0.72667984189723323; macroAverageRecall=0.73690515532055523; 
                   matthewsCorrelationCoefficient=0.46347221341824979}
links            : {@{rel=self; href=https://ml.nexosis.com/v1/models/1b79d672-99a4-47f7-a387-a001d9420220}, @{rel=train; 
                   href=https://ml.nexosis.com/v1/sessions/0160ddce-965a-4281-8fd7-6a58e1cbba4b}}</code>
</pre>
    </div>
</div>

Focus in on the `algorithm` and `metrics` nodes in the JSON response. The algorithm used is a `Support Vector Classification use Radial Basis Function` which resulted in an accuracy score is 72% for this model. As was mentioned already, providing more data which building the model can increase the accuracy.


<ul id="profileTabs" class="nav nav-tabs">
    <li class="active"><a href="#curlModelAlgoMetrics" data-toggle="tab">Curl</a></li>
    <li><a href="#powershellModelAlgoMetrics" data-toggle="tab">Powershell</a></li>
</ul>
<div class="tab-content">
    <div role="tabpanel" class="tab-pane active" id="curlModelAlgoMetrics">
    <p>The following command assumes your Nexosis API Key is stored in an Environment Variable called <code>$NEXOSIS_API_KEY</code>.</p><pre class="language-json"><code class="language-json code-toolbar">{
    // ...elided
    "algorithm": {
        "name": "SVC RBF",
        "description": "Support Vector Classification using Radial Basis Function (Gaussian) kernel",
        "key": "svc_rbf"
    },
    "metrics": {
        "macroAverageF1Score": 0.72521972411860092,
        "rocAreaUnderCurve": 0.79444811632518164,
        "accuracy": 0.72888888888888892,
        "macroAveragePrecision": 0.72667984189723323,
        "macroAverageRecall": 0.73690515532055523,
        "matthewsCorrelationCoefficient": 0.46347221341824979
    },
    // ...elided
}</code></pre>
   </div>
   <div role="tabpanel" class="tab-pane" id="powershellModelAlgoMetrics">
      <p>The following command assumes your Steam API Key is stored in an Environment Variable called <code>$NEXOSIS_API_KEY</code>.</p>
      <pre class="language-powershell"><code class="language-powershell code-toolbar">PS> $modelDetail = Get-NexosisModelDetail -ModelId 20a41d0e-3663-469b-bdc0-5e2e1e12e0e4
PS> $modelDetail.algorithm

name    description                                                                 key    
----    -----------                                                                 ---    
SVC RBF Support Vector Classification using Radial Basis Function (Gaussian) kernel svc_rbf

PS> $modelDetail.metrics

macroAverageF1Score            : 0.72521972411860092
rocAreaUnderCurve              : 0.79444811632518164
accuracy                       : 0.72888888888888892
macroAveragePrecision          : 0.72667984189723323
macroAverageRecall             : 0.73690515532055523
matthewsCorrelationCoefficient : 0.46347221341824979

</code>
</pre>
    </div>
</div>


## Using the Model

Now that we have a model, let's use it to predict. We're going to need to retrieve data on a player from the Steam API, calculate the accuracy stats, and then submit it to the model prediction endpoint in the Nexosis API using the Model ID assigned to this model.

To check to see how well our model did, I did some googling and found some complaints against certain Steam accounts and accused them of cheating so I ran them through this model and I also used vac-ban.com as well to find other already banned accounts to help validate the model.

Here's the script using a function I wrote called `Invoke-CalculateCsGoStatsForSteamId` that does the Steam API call and calcuates the statistics and packages the data up properly to be submitted to the API:

<pre class="language-powershell" style="max-height:38em;"><code class="language-powershell code-toolbar">$suspectStatData = @()
# suspected cheaters identified by other players - posted in different forums online
$suspectedCheatSteamID = @(
    '76561198361486862',  # Steam ID of suspected cheater - stats indicate cheater
    '76561198121540097',  # http://www.vac-ban.com/76561198121540097/stats.html - someone mentioned suspicion in a forum - stats don't trip cheat detection
    '76561197978008587',  # http://www.vac-ban.com/76561197978008587/stats.html
    '76561198097618775',  # http://www.vac-ban.com/76561198097618775/stats.html
)

foreach ($steamId in $suspectedCheatSteamID) {
    $suspectedCheatStats = Invoke-CalculateCsGoStatsForSteamId($steamId)
    if ($suspectedCheatStats -ne $null) {
        $suspectStatData += $suspectedCheatStats
    }
    Start-Sleep -Milliseconds 500
}

# Run prediction using Model
$suspiciousResults = Invoke-NexosisPredictTarget -modelId 1b79d672-99a4-47f7-a387-a001d9420220 -data $suspectStatData -Verbose
$suspiciousResults.data
</code></pre>

Additionally, I captured the `VACBanned` flag from the Steam API and re-named it `VACBannedActual`. The Nexosis API will reflect back any column submitted in a prediction, but will not use unrecognized elements - this way we'll get back a `VACBanned` prediction from the Nexosis API and VacBannedActual which is what the Steam API has on record for this player.

Here is the RAW JSON body needs to be packaged up as a JSON array like so in a `POST` to `https://ml.nexosis.com/v1/models/{modelId}/predict`

<pre class="language-json" style="max-height:30em;"><code class="language-json code-toolbar">{
    "data":  [
      {
        "accuracy_mp7":  0.17872340425531916,
        "accuracy_hkp2000":  0.18116805721096543,
        "accuracy_glock":  0.157167963846347,
        "accuracy_elite":  0.17216642754662842,
        "accuracy_ssg08":  0.34361233480176212,
        "accuracy_m4a1":  0.21935575826681869,
        "accuracy_ump45":  0.15429791777896423,
        "total_accuracy":  0.173028144228552,
        "accuracy_m249":  0.051560379918588875,
        "accuracy_aug":  0.20770600615185364,
        "accuracy_g3sg1":  0.33160621761658032,
        "kill_to_death_ratio":  3.136818959198632,
        "accuracy_p90":  0.15919519943522767,
        "accuracy_mag7":  0.18167938931297709,
        "total_headshots_per_round":  0.44644779332615714,
        "accuracy_sg556":  0.15061475409836064,
        "accuracy_fiveseven":  0.08387096774193549,
        "mvp_per_round":  0.10656620021528525,
        "accuracy_negev":  0.033666255190214343,
        "accuracy_tec9":  0.17585301837270342,
        "accuracy_scar20":  0.26609442060085836,
        "total_wins_per_hour":  5.7162284075067678E-06,
        "VACBannedActual":  0,
        "steamId":  "76561198361486862",
        "accuracy_sawedoff":  0.12444444444444444,
        "accuracy_p250":  0.2,
        "accuracy_nova":  0.24717145343777197,
        "accuracy_xm1014":  0.20195960807838431,
        "accuracy_mp9":  0.168359375,
        "accuracy_ak47":  0.16113012549493322,
        "total_games_owned":  301,
        "accuracy_awp":  0.43594386600271617,
        "accuracy_famas":  0.18745785569790965,
        "accuracy_deagle":  0.25171467764060357,
        "win_ratio":  1.711248654467169,
        "accuracy_galilar":  0.17252283907238228,
        "accuracy_bizon":  0.20948616600790515,
        "accuracy_mac10":  0.1984271022383545
      },
      {
        "accuracy_mp7":  0.20782892352301557,
        "accuracy_hkp2000":  0.18866790009250695,
        "accuracy_glock":  0.15199442419043535,
        "accuracy_elite":  0.16281774381126432,
        "accuracy_ssg08":  0.28807106598984772,
        "accuracy_m4a1":  0.19933089278520258,
        "accuracy_ump45":  0.17035325189831627,
        "total_accuracy":  0.19232434737353687,
        "accuracy_m249":  0.075642111205193333,
        "accuracy_aug":  0.18128639421415088,
        "accuracy_g3sg1":  0.24681271194928955,
        "kill_to_death_ratio":  1.2476448050484561,
        "accuracy_p90":  0.12632232247068195,
        "accuracy_mag7":  0.20981713185755535,
        "total_headshots_per_round":  0.62682415176942718,
        "accuracy_sg556":  0.16531428571428572,
        "accuracy_fiveseven":  0.18956853231675619,
        "mvp_per_round":  0.12531922655964975,
        "accuracy_negev":  0.045550905038865032,
        "accuracy_tec9":  0.16504026527711985,
        "accuracy_scar20":  0.29138178561783057,
        "total_wins_per_hour":  1.3370898632388331E-06,
        "VACBannedActual":  0,
        "steamId":  "76561198121540097",
        "accuracy_sawedoff":  0.16362204724409449,
        "accuracy_p250":  0.22663152999534955,
        "accuracy_nova":  0.19207501512401695,
        "accuracy_xm1014":  0.14102984585109873,
        "accuracy_mp9":  0.17253021811060842,
        "accuracy_ak47":  0.16937251453545601,
        "total_games_owned":  3,
        "accuracy_awp":  0.44058280028429281,
        "accuracy_famas":  0.15850956696878146,
        "accuracy_deagle":  0.25248592623770189,
        "win_ratio":  0.51942721634439981,
        "accuracy_galilar":  0.15838808585194919,
        "accuracy_bizon":  0.14906930693069306,
        "accuracy_mac10":  0.18077354260089687
      },
      {
        "accuracy_mp7":  0.16457399103139014,
        "accuracy_hkp2000":  0.22922909289022064,
        "accuracy_glock":  0.18168002210555403,
        "accuracy_elite":  0.14909090909090908,
        "accuracy_ssg08":  0.35866614967041488,
        "accuracy_m4a1":  0.15684093437152391,
        "accuracy_ump45":  0.13016845329249618,
        "total_accuracy":  0.21874758825072932,
        "accuracy_m249":  0.12008733624454149,
        "accuracy_aug":  0.15062761506276151,
        "accuracy_g3sg1":  0.077762619372442013,
        "kill_to_death_ratio":  0.10682981229220298,
        "accuracy_p90":  0.50327966607036378,
        "accuracy_mag7":  0.18232454511696991,
        "total_headshots_per_round":  0.46772655007949127,
        "accuracy_sg556":  0.079805056350898573,
        "accuracy_fiveseven":  0.27650727650727652,
        "mvp_per_round":  0.059565447800741918,
        "accuracy_negev":  0.81597021100537859,
        "accuracy_tec9":  0.16871165644171779,
        "accuracy_scar20":  1.1100917431192661,
        "total_wins_per_hour":  8.0417772278010944E-07,
        "VACBannedActual":  1,
        "steamId":  "76561197978008587",
        "accuracy_sawedoff":  0.14627241270839886,
        "accuracy_p250":  0.23512441399206635,
        "accuracy_nova":  0.19236957021374396,
        "accuracy_xm1014":  0.18820468343451865,
        "accuracy_mp9":  0.13748191027496381,
        "accuracy_ak47":  0.15761821366024517,
        "total_games_owned":  14,
        "accuracy_awp":  0.4609004739336493,
        "accuracy_famas":  0.17543335325762105,
        "accuracy_deagle":  0.46508771929824561,
        "win_ratio":  0.48055113937466881,
        "accuracy_galilar":  0.10905587668593449,
        "accuracy_bizon":  0.16933430338304839,
        "accuracy_mac10":  0.13968715896689704
      },
      {
        "accuracy_mp7":  0.23970290344361916,
        "accuracy_hkp2000":  0.26665804950559041,
        "accuracy_glock":  0.21003454773869346,
        "accuracy_elite":  0.22915011914217634,
        "accuracy_ssg08":  0.52289377289377292,
        "accuracy_m4a1":  0.23553711455153631,
        "accuracy_ump45":  0.20850265708033761,
        "total_accuracy":  0.21440652294643309,
        "accuracy_m249":  0.24479804161566707,
        "accuracy_aug":  0.27552275522755226,
        "accuracy_g3sg1":  0.19650391802290537,
        "kill_to_death_ratio":  1.6247155747351902,
        "accuracy_p90":  0.17222795550968925,
        "accuracy_mag7":  0.2653876898481215,
        "total_headshots_per_round":  0.52306958358890476,
        "accuracy_sg556":  0.20404814004376368,
        "accuracy_fiveseven":  0.29589041095890412,
        "mvp_per_round":  0.16826824780208546,
        "accuracy_negev":  0.097546556310966592,
        "accuracy_tec9":  0.20973544973544975,
        "accuracy_scar20":  0.37002652519893897,
        "total_wins_per_hour":  2.06431806294603E-06,
        "VACBannedActual":  1,
        "steamId":  "76561198097618775",
        "accuracy_sawedoff":  0.1980881571959639,
        "accuracy_p250":  0.26325284485151262,
        "accuracy_nova":  0.29329102447869448,
        "accuracy_xm1014":  0.26422602467170714,
        "accuracy_mp9":  0.24713958810068651,
        "accuracy_ak47":  0.18438532429142757,
        "total_games_owned":  278,
        "accuracy_awp":  0.51533301389554387,
        "accuracy_famas":  0.21656199116073505,
        "accuracy_deagle":  0.33237867939538585,
        "win_ratio":  0.56109861650650861,
        "accuracy_galilar":  0.20742278110120865,
        "accuracy_bizon":  0.19746208604147322,
        "accuracy_mac10":  0.19654817104585265
      }
    ]
}
</code></pre>

The response back from the Nexosis API will contain all the data that was submitted and it will add the prediction target `VACBanned`. Here's the response to our predictions - when comparing `VACBanned` with `VACBannedActual` this model agrees with 3 of the 4 Steam ID's below.

<pre class="language-json" style="max-height:30em;"><code class="language-json code-toolbar">{
    "data": [{
            "steamId": "76561198361486862",
            "VACBannedActual": "0",
            "VACBanned": "0",
            // data elided
        },
        {
            "steamId": "76561198121540097",
            "VACBannedActual": "0",
            "VACBanned": "1",
            // data elided
        }, {
            "steamId": "76561197978008587",
            "VACBannedActual": "1",
            "VACBanned": "1",
            // data elided
        }, {
            "steamId": "76561198097618775",
            "VACBannedActual": "1",
            "VACBanned": "1",
            // data elided
        }
    ],
    "messages": [{
        "severity": "warning",
        "message": "Column 'VACBannedActual' was not a feature in the training data for this model, so it will not be considered for prediction."
    }, {
        "severity": "warning",
        "message": "Column 'steamId' was not a feature in the training data for this model, so it will not be considered for prediction."
    }],
    "modelId": "1b79d672-99a4-47f7-a387-a001d9420220",
    "sessionId": "0160ddce-965a-4281-8fd7-6a58e1cbba4b",
    "predictionDomain": "classification",
    "dataSourceName": "csgo-stats",
    "createdDate": "2018-01-10T02:06:06.6928748+00:00",
    "algorithm": {
        "name": "SVC RBF",
        "description": "Support Vector Classification using Radial Basis Function (Gaussian) kernel",
        "key": "svc_rbf"
    },
    "metrics": {
        "macroAverageF1Score": 0.72521972411860092,
        "rocAreaUnderCurve": 0.79444811632518164,
        "accuracy": 0.72888888888888892,
        "macroAveragePrecision": 0.72667984189723323,
        "macroAverageRecall": 0.73690515532055523,
        "matthewsCorrelationCoefficient": 0.46347221341824979
    },
    "links": []
}
</code></pre>

## Final Thoughts: Improving the Model

There's certainly more to do to improve this model but our first pass has us off to a great start. As has been mentioned a few times before, using more data will help improve the accuracy of the model. But not only is more data required, but the right data - this takes some thought about the problem and experimatiation.

It's very likely our model has some limitations based on the data available from the Steam API. For example, we can't take into account a players performance per game map which might help improve the model, or per game mode (CSGO has 13 different modes) since the Steam API doesn't provide stats per game mode. For example, if someone tends to play less common game mode like Capture and Hold, it may influence the models ability to accurately predict for them. With enough data hopefully these types of differences can be represented in the model, thus improving the accuracy. 

Finally, our follow-up research indicates that the more important factors that could be used to identify cheaters and using these features instead can help us build a much more accurate model:

1. Weapons donated per hour
2. Total time played
3. P250 Shot ratio 
4. P250 gun kill ratio
5. P25 gun hit ratio

When these were used we achieved an accuracy of just over 90%.  How well do you think you can do?

Have fun experimenting and let us know!