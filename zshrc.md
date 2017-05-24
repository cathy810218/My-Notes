
```
openxc() {
    local project="$(ls | grep '.*\.xcworkspace$')"
    [[ -z $project ]] && project="$(ls | grep '.*\.xcodeproj$')"
    if [[ -n $project ]]; then
        open $project
    else
        echo "No .xcworkspace or .xcodeproj files in current directory" >&2
        return 1
    fi
}


alias sb="'/Applications/Sublime Text.app/Contents/SharedSupport/bin/subl'"
alias zshconfig='subl ~/.zshrc'
alias dev='cd Developer'
alias gcm='git checkout master'
```