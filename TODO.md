# TODO

- Have an option for each match to tell you the exact sentence index span that the match found in the original sentence (not the linear tree). This will require a lot more work, as the regex matching happens on the _linear tree_, not the original sentence 

- The \<word\> part of the linear tree schema (\<word\>-\<lemma\>-\<pos\>-\<deplink\>) may not be needed. Could be removed, which would make the regex slightly more readable (and possibly optimize it a bit?). I say this because it has not played a part in any of the regex implemented yet.

- Add another layer of matching to help with more complex syntactic patterns.

# WIP

The following patterns are WIP:

- dependent-clauses https://regex101.com/r/TgWuWz/1

- yes-no-questions https://regex101.com/r/ATqtWh/1

- wh-questions  https://regex101.com/r/uJ7WEy/1