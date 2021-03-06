## What the package does

Package to simplify using opendota.com's API, get game lists, and download JSON's of parsed replays 
from the opendota API. Also has functionality to execute own code to extract the specific parts of 
the JSON file. This has no direct relation to opendota.com, i'm just a guy that likes playing with
their data and needed a neat wrapper to fetch it more efficiently :)

## How to use the package

There are two basic ways of using the package, either to get the total JSON for each replay, which
is the simple way of doing it as it requires no changes, the other way is to create your own .R file
that reads the JSON (variable read_json) and selects specific parts of it to output.

### Simple version
To get the full JSON output, just put a list of match_id's (in numeric vector form) into get_games(),
to get a list of games, you can use get_game_list(). Note that only about 5% of matches are parsed,
and higher num_open_profile (number of players in game with an open profile) will increase your
percentage of parsed matches.

```R
game_list <- get_game_list(num_matches = 100,
                           from_time = "20170401",
                           to_time = "20170425",
                           min_mmr = 3000,
                           num_open_profile = 6)

match_id_vec <- game_list$match_id

parsed_games <- get_games(match_id_vec)
```

Output will be a list, each list item will be the JSON from a parsed replay.

### More complex version
Each parsed mach is read into a read_json variable in the R function environment, and then output to
output_list if output = "all", if however you specify an R file destination in output, for example
output = "test.R", it will source that R file for each parsed game.

That means you will have to create some output and make sure it gets assigned to output_list like so

```R
output_list[[parsed_count]] <- your_output
```

You will also need to load all libraries needed inside the R, file, here is an example that outputs
the hero_id, and number of wards bought in the game. I save the file as test.R

```R
library(jsonlite)
library(data.table)

ward_list <- list()

# Iterate through each player, getting the values we need.
for(j in 1:10) {
  # Total number of wards bought the first 20 minutes
  purchase_log <- as.data.frame(read_json$players$purchase_log[j])
  
  if (nrow(purchase_log) > 0) {
    wards_only <- subset(purchase_log, 
                         (purchase_log$key == "ward_sentry" | purchase_log$key == "ward_observer"))
    
    if (nrow(wards_only) > 0) {
      wards_bought <- data.frame(hero = read_json$players$hero_id[j],
                                 wards_bought = nrow(wards_only))
    } else {
      wards_bought <- data.frame(hero = read_json$players$hero_id[j],
                                 wards_bought = 0)
    }
  } else {
    wards_bought <- data.frame(hero = read_json$players$hero_id[j],
                               wards_bought = 0)
  }
  
  ward_list[[j]] <- wards_bought
}

output_list[[parsed_count]] <- rbindlist(ward_list, fill = TRUE)

```

If you know just call get_games() the same way as before, but using output = "test.R" (or whereever
you've stored the R file), you will get list output with the total number of wards bought by each 
hero in all games parsed.

```R
game_list <- get_game_list(num_matches = 100,
                           from_time = "20170401",
                           to_time = "20170425",
                           min_mmr = 3000,
                           num_open_profile = 6)

match_id_vec <- game_list$match_id

parsed_games <- get_games(match_id_vec, output = "test.R")
```

Note the output here will be a list of multiple data frames, to get it as one large dataframe simply
use rbindlist(parsed_games).

### Get only the latest games
Using the get_game_list() you can specify specific times, minimum MMR and some other criteria to get
a more specific list of games, but this will leave you with a list where only 5-10% of your games 
are parsed, so that´s a lot of wasted API calls, if you're lucky 1 in 10 will give you a parsed games,
and with the delay of 1 second, you get one game every second.

So if you are OK with only getting the latest games, no filtering on MMR or when the game was played,
you can use the get_latest_games() function, which guarantees a 100% hitrate, giving you a dataset to
work with a lot faster than the get_game_list() and get_games() combo.

It works very simply, it queries https://api.opendota.com/api/status which gives the 10 latest added
games, and parses those. Within the function it still uses get_games() so you can still use the 
output variable to specify an R file that prunes the JSON and only outputs what you're interested in.

This would give you the full JSON output for the past 100 games
```R
parsed_games <- get_latest_games(100)
```

While this would give you the latest 100 games, but only outputting whatever test.R specifies (see
code example above where we output heroID and number of wards bought).
```R
parsed_games <- get_latest_games(100, output = "test.R")
```
Many thanks to [Howard Chung](https://github.com/howardchung) (one of main contributes to the opendota project) for pointing out the 
API status call and that it might be helpful for this package.

## Installing
The package is available on cran, so install.packages("opendotaR"), or if you're using RStudio, clicking on
the "Packages" tab, and "Install" button there, write in opendotaR and it should install without issues.
