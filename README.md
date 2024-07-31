# hng11_ansible_task
An nextjs boilerplate

WARNING  Listing 11 violation(s) that are fatal
yaml: truthy value should be one of [false, true] (truthy)
main.yaml:4

yaml: trailing spaces (trailing-spaces)
main.yaml:5

yaml: truthy value should be one of [false, true] (truthy)
main.yaml:11

yaml: trailing spaces (trailing-spaces)
main.yaml:41

yaml: truthy value should be one of [false, true] (truthy)
main.yaml:46

risky-file-permissions: File permissions unset or incorrect
main.yaml:48 Task/Handler: Setup Nginx to reverse proxy application to port 80

yaml: trailing spaces (trailing-spaces)
main.yaml:53

risky-file-permissions: File permissions unset or incorrect
main.yaml:54 Task/Handler: Create log directory

risky-file-permissions: File permissions unset or incorrect
main.yaml:61 Task/Handler: Configure stderr logging

risky-file-permissions: File permissions unset or incorrect
main.yaml:68 Task/Handler: Configure stdout logging

yaml: no new line character at the end of file (new-line-at-end-of-file)
main.yaml:73

You can skip specific rules or tags by adding them to your configuration file:
# .ansible-lint
warn_list:  # or 'skip_list' to silence them completely
  - experimental  # all rules tagged as experimental
  - yaml  # Violations reported by yamllint