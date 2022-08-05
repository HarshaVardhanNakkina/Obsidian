## [Data flow](https://docs.streamlit.io/library/get-started/main-concepts#data-flow)
- Every time you want to update your app, save the source file. When you do that, Streamlit detects if there is a change and asks you whether you want to rerun your app. Choose "Always rerun" at the top-right of your screen to automatically update your app every time you change its source code.

Streamlit's architecture allows you to write apps the same way you write plain Python scripts. To unlock this, Streamlit apps have a unique data flow: any time something must be updated on the screen, Streamlit reruns your entire Python script from top to bottom.

This can happen in two situations:
- Whenever you modify your app's source code.
- Whenever a user interacts with widgets in the app. For example, when dragging a slider, entering text in an input box, or clicking a button.

Whenever a callback is passed to a widget via the `on_change` (or `on_click`) parameter, **the callback will always run before the rest of your script.** For details on the Callbacks API, please refer to our [Session State API Reference Guide](https://docs.streamlit.io/library/api-reference/session-state#use-callbacks-to-update-session-state).

## [Display and style data](https://docs.streamlit.io/library/get-started/main-concepts#display-and-style-data)
There are a few ways to display data (tables, arrays, data frames) in Streamlit apps. [Below](https://docs.streamlit.io/library/get-started/main-concepts#use-magic), you will be introduced to _magic_ and [`st.write()`](https://docs.streamlit.io/library/api-reference/write-magic/st.write), which can be used to write anything from text to tables. After that, let's take a look at methods designed specifically for visualizing data.

### [Use magic](https://docs.streamlit.io/library/get-started/main-concepts#use-magic)
You can also write to your app without calling any Streamlit methods. Streamlit supports "[magic commands](https://docs.streamlit.io/library/api-reference/write-magic/magic)," which means you don't have to use [`st.write()`](https://docs.streamlit.io/library/api-reference/write-magic/st.write) at all! To see this in action try this snippet:
``` python
"""
# My first app
Here's our first attempt at using data to create a table:
"""

import streamlit as st
import pandas as pd
df = pd.DataFrame({
  'first column': [1, 2, 3, 4],
  'second column': [10, 20, 30, 40]
})

df
```
Any time that Streamlit sees **a variable or a literal value on its own line**, it automatically writes that to your app using [`st.write()`](https://docs.streamlit.io/library/api-reference/write-magic/st.write). For more information, refer to the documentation on [magic commands](https://docs.streamlit.io/library/api-reference/write-magic/magic).