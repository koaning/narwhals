# DataFrame Interoperability

I attended PyConLithuania 2024, and had a blast! I don't speak Lithuanian, and probably
neither did half the attendees. But - so long as we all stuck to a simple and clear subset
of the English language - we could all understand each other and exchange ideas.

Dataframes, like languages, have their similarities and differences. If you learn a bit of
Spanish, you might think that "estoy embarazado" means "I'm embarrassed", but it actually
means "I'm pregnant". Similarly, after learning a bit of pandas and Polars, you might
expect
```python
import pandas as pd
import polars as pd

print((3 in pd.Series([1,2,3])) == (3 in pl.Series([1,2,3])))
```
to print `'True'` - but no so! pandas checks if `3` is in the index, whereas Polars checks
if `3` is in the values.

This little prelude was just to establish two premises:

- writing dataframe-agnostic code is hard!
- a simple and clear common language can enable collaboration

## pandas are everywhere

There's a great array of diverse dataframes out there. And the way that data science libraries
have typically to such diversity is to support pandas.

[image]

This is nice, but comes with four major problems:

- for non-pandas users, they're forced to repeatedly convert to-and-from pandas, which isn't
  ergonomic;
- for users starting with data on GPU, they need to copy to CPU;
- users working with lazy evaluation need to materialise their data;
- pandas become a de-facto required dependency everywhere.

Before talking about how to solve all 4, let's talk about how to solve at least one of them.

## The Dataframe Interchange Protocol

The general idea is "write your library using dataframe X, and then if a user comes along with
dataframe Y, then you use the interchange protocol to convert dataframe X to dataframe Y".

[image]

Let's look at how to use it - here's how you can convert any dataframe to pandas
(so long as it implements the interchange protocol):

```python
import pandas as pd

df_pandas = pd.api.interchange.from_dataframe(df_any)
```

Similarly, to convert to Polars:

```python
import polars as pl

df_pandas = pl.from_dataframe(df_any)
```

Note that although the `from_dataframe` function is standardised,
where it appears in the API isn't, so there's no completely agnostic way
of round-tripping back to your starting dataframe class.

Nonetheless, does it work? Is it reliable?

- Converting to pandas: reliable enough after pandas 2.0.2, though may sometimes
  raise unnecessarily in some versions. It's already used by the `seaborn` and
  `plotly` dataframe libraries.
- Converting from pandas: unreliable, don't use it. There are cases when the results
  are literal nonsense. These have generally been fixed, and will be available in
  pandas 3.0+. For now, however, if you're using the interchange protocol to convert
  to anything other pandas, then you may want to proceed with great care.

So as of April 2024, the interchange protocol can be used as a standardised version
of `to_pandas`. Going back to the four problems with only supporting pandas, the
interchange protocol partially addresses the first one (so long as the user doesn't
expect their computation to round-trip back to their original class), but not the others.

Addressing all four points seems ambitious - is it even possible?

## And now for something completely different: Narwhals

Look at how happy this panda and polar bear looking, chilling out with their narwhal friend:

[image]

As you may have guessed, Narwhals aims to bring pandas and Polars together.
