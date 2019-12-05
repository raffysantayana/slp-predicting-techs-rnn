<a id='problem'></a>
## Problem Statement

Note: For those unfamiliar with Super Smash Bros., I encourage you to read the [context](#context) section.

Fox versus Falco is a frequent match in competitive Melee. Discussions about whether Fox or Falco have the upperhand in the matchup is a popular debate among players. I believe that among today's top ranked players in the world, Fox will most likely win because of his faster speed over Falco. However, Falco is more likely to win against newer players because he is able to utilize different tools to perform combos that lead to Fox losing a stock.

Since Falco is often one of the first major hurdles for new players to defeat, I hope to construct a coaching tool that will help them overcome this. The final product will use a collection of Slippi games as input and provide summary statistics on the games. For coaching tools of the product, a player can feed in a collection of Slippi games to learn how a specified player and character plays. Then this will create an AI that someone can battle. This is useful for those who wish to practice against a live player, but does not have immediate access to others. A similar idea to how Super Smash Bros. Wii U utilized [Amiibos](https://www.youtube.com/watch?v=uOnLcVOvrEE). As a quick check to see how feasible this project is, I will train a recurrent neural network on a series of games to see how well the network can learn Fox's wake-up behavior.
<p align = "center">
  <img src="images/melee-wallpaper.jpg" alt="Drawing" style="width: 600px;"/>
</p>

<a id='context'></a>
## Context
<a id='why'></a>
### Why Super Smash Bros.?

Super Smash Bros. is a video game series published by Nintendo where video game characters of different franchises battle it out. The second game of the series, Super Smash Bros. Melee, was released in December 2001. This sparked fun parties, happy players, and heated rivalries. These rivalries grew from friend groups, to local neighborhood challenges, to large-scale tournaments by 2002. If you are interested in learning the history of the Super Smash Bros. Melee's competitive scene, I encourage you to watch a docuseries called [The Smash Brothers](https://www.youtube.com/watch?v=NSf2mgkRm7Q&list=PLoUHkRwnRH-IXbZfwlgiEN8eXmoj6DtKM) produced by East Point Pictures.

Many players within this community are dedicated to becoming the best player that they can be. There are many fan-made tools and mods to the game for the purpose of either improving the player experience in training or improving the production of content. Some tools allow players to create save states so that they can easily practice a scenario quickly. Others allow content creators to create replays for the audience to enjoy. Today, I would like to highlight one of these tools, Project Slippi.
- [Website](https://slippi.gg/)
- [Github](https://github.com/project-slippi/project-slippi)
- [Medium](https://medium.com/project-slippi)

<a id='how'></a>
### How Does Super Smash Bros. Work?

In this project, I will be only looking at 1v1 tournament legal matches. The criteria for that are:
- The match must be 1v1 with no other players or CPU's in the match.
- The match must be on a tournament legal stage.
- The match must be 8 minutes or less.

In a 1v1 match, players are limited to defeat their opponent with nothing more than the abilities of themselves as a player and the abilities of their selected character. Characters have various moves such as jabs, smash attacks, tilt attacks, special attacks, grabs, and aerial attacks. Most of these attacks have slight variations of themselves, but at different directions. When a player strikes their opponent with an attack, then the opponent's damage percentage goes up.
<p align = "center">
  <img src="images/damage-example.gif"/>
  <b>Fox Damaging Falco</b>
</p>

As a character's damage increases, then the distance at which they are launched after a hit then increases. This is beneficial as an opponent because the further they travel for each hit, then it should be easier to push them through a blast zone and have them lose a stock. Once a character loses all their stocks, then the other player is determined the winner. If a timeout occurs, then the player with the most stocks and least damage wins. In the case that those are tied as well, then a rematch is played with each charcter getting one stock.
<p align = "center">
  <img src="images/GAME.gif"/><br>
  <b>Jigglypuff Defeating Fox's Last Stock; Winning the Game</b>
</p>

### A Primer on Fighting Games

Fighting games is one of the many genres of video games. Some fighting game titles that may sound familiar are Street Fighter and Tekken. These games are unique in that Street Fighter is a 2-dimensional fighter where characters can only move left or right and Tekken characters can move forward, backwards, left or right. While all of these games are systematically different, they all share similar strategies that a player can utilize. For example, players can play as a heavy brute that can pack a punch, but are slow. Or they can play a glass cannon that can attack with lightning fast speed, but they are easy to kill once their momentum is broken.

The idea of mixups is another common thread among fighting games. One of the most frequent mixup situations that occur in fighting games are when a character is laying on the ground. Usually, the character has one of four options known as get-up options or wake-up options. They can stay on the ground, get up, roll forward, or roll back. The opponent must be able to either predict or quickly react to whichever option the player takes and act accordingly. If the opponent does not capitalize on the player's vulnerability, then the player has a chance to fight back and win.

In Melee, when a character is launched towards the floor, wall, or ceiling, then they have the option to tech. When a player is hit and placed in a situation where they must decide whether to tech in-place, tech-roll left, or tech-roll right, they may miss the 20 frame window and not tech. This will cause them to stay vulnerable on the ground with few options to retaliate. However, like stated before, this is a mixup. If a player frequently techs to the right, then their opponent will be accustomed to that behavior. So whenever the the opponent sees that the player has the option to tech, then they will assume the player will tech right and adjust the combo. If the player suddenly techs in-place and the opponent did not expect that, then the player has recovered safely and is free to move. This process is known as tech chasing - the opponent is tech-chasing the player.
<p align = "center">
  <img src="images/tech-chase.gif"/><br>
  <b>Captain Falcon tech-chasing Fox McCloud</b>
</p>


<a id='executive'></a>
## Executive Summary
<a id = 'gather'></a>
### Data Gathering
To begin, I considered constructing a script that would scrape [Slippi's site](https://slippi.gg) for games that occured during a tournament using the library `selenium`. I opted not to do this because the official Slippi Discord channel has the `!replaydumps` chat command that provides a download link to Slippi files from [Fight Pitt 9](https://smash.gg/tournament/fight-pitt-9-1/details), [Full Bloom 5](https://smash.gg/tournament/full-bloom-5/details), [The Gang Steals the Script](https://smash.gg/tournament/the-gang-steals-the-script/details), and [Pound 2019](https://smash.gg/tournament/pound-2019/details). The source of the data is on a different platform, but each are controlled by the creators and major contributors to the project such as [Fizzi](https://twitter.com/Fizzi36).

<a id='parse'></a>
### Parsing Data
With the data in hand, I used the `slippi` library to read in each file as a Slippi's own Game object. Game objects have attributes whose values are sometimes other objects. Since each Slippi file has the same structure, I created a function `metadata_to_df` to parse the metadata of each game. The objective here is to filter the games as needed. For example, I want 1v1 games, so I can filter for games where the team battle option is set to off.

<a id = 'modeling'></a>
### Modeling
Once I have filtered the games I will use as input, I will then create another function to parse the Frame objects of each game. Each Frame object contains information of each frame within the game such as character position and controller inputs. These values will be fed to a recurrent neural network (RNN). The RNN will have a simple topology of a single hidden layer. This is because I am interested to see how well the RNN can learn the players behavior the least amount of complexity possible. If the results were not much better than the baseline accuracy score, then I would increase the complexity of the topology, but cannot due to the below limitations.

<a id = 'limitatdoesnotexist'></a>
### Limitations
tl;dr Power and money.

When parsing the metadata and frame data from each game, it took a considerable amount of time to execute for all Fight Pitt 9 games. This was a concern because Fight Pitt 9 contained the least amount of games compared to the other tournaments. This lack of power encouraged me only utilize Fight Pitt 9 games.

Since every frame of a game contains multiple features that can be used as inputs to the RNN, there is a lot of data to deal with. If a single game lasts for a minute, then that is 3600 frames (60 seconds * 60 frames per second). Each frame contains 333 features after cleaning and dummying. In short, a lot of power is needed to be able to fit the neural network quickly.

I could use AWS cloud computing to perform the task, but the machine's that were noticeably stronger than my machine cost too much for me at the moment.
