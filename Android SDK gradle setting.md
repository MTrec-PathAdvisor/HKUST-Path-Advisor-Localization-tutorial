The Wherami Android SDK package is hosted on http://43.252.40.61:8081/repository/wherami/ .
Android Apps may use the following gradle setting in your project-level build.gradle.

example:
```maven
maven {
            url "http://43.252.40.61:8081/repository/wherami/"

            credentials {
                username 'path_advisor'
                password 'hkust'
            }
        }
```
