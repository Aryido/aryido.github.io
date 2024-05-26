## Domain Name
https://aryido.github.io/


## Install
To download the theme, use:
```
git submodule update --init
```

Note:
- Due to the `Hugo Tranquilpeak theme` no longer being updated, it is recommended to use Hugo version `0.119.0`, or fix the original code from submodule:
  ```
  {{ template "_internal/google_analytics_async.html" . }}
  
  >>
  
  {{ template "_internal/google_analytics.html" . }}

  ```

## local mod
```
hugo server -D
```

## Reference
https://github.com/kakawait/hugo-tranquilpeak-theme