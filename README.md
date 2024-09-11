# PR topic_pool [![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)

PR topic_pool use GitHub workflow/action/repo to implement [gerrit-style topic](https://gerrit-review.googlesource.com/Documentation/intro-user.html#topics) function.

The code is split across multiple repositories in larger code bases, when making a change, you might make code changes that span multiple repositories. Submitting these changes across these repositories separately could cause the build to break for other developers. PR topic to submit changes together to prevent this problem. A topic is a string that can be associated with a change. Multiple changes can use that topic to be submitted at the same time from different repos.

#### Changelog:<br>
20230110: Design for gerrit to GitHub migration<br>
20240724: Re-design for remove Jenkins dependencies<br>
20240731: Opensource version Release<br>

### Design Philosophy
- Small is beautiful
- Store data in flat text files
- Use shell scripts to increase leverage and portability
- Version control for every change

### Feature list
- PR topic
  - The topic can only have one PR in the same repository
  - customize topic name with [TOPIC=topic_name] in git commit message
  - if use JiraID:88888 or JiraCloudID:99999 in git commit message and no topic define, the topic name will be 70P1C-JID-88888 or 70P1C-JCID-99999
  - if no topic define and any Jira id in git commit message, the default topic name use current PR URL hash like 70P1C-55783a3
  - same repo's PR replacement detection and notification in topic
  - check that the topic name complies with the naming convention
- topic list
  - Use @topic_list in PR comment get PRs list which in same topic
- Open interface for @topic_build, @topic_merge, @topic_revert
- topic clean
  - Automatically clean up topic branches older than 90 days at 00:00 every Sunday 
- Data structure
  - topic info storge in topic branch
  - PR info storge in repo name file
  - Content examples:
```
TP_GH_REPO=ORG/REPO
TP_JIRA_CLOUD_ID=
TP_JIRA_ID=88888
TP_PR_BASE_BRANCH=main
TP_PR_HEAD_BRANCH=sandbox/demo_for_topic
TP_PR_HEAD_GIT_LOG=Demo_for_Topic
TP_PR_HEAD_SHA=234f624c72f0f1a2de12342a0cdb98b4e113c856
TP_PR_NODE_ID=PR_kwDOLdGgCM91r1j0
TP_PR_URL=https://github.com/ORG/REPO/pull/9999
```
![topic pool design](/topic_pool.png)

### How to implement?
1. Create a new topic pool repo to store topic information named PR_topic_pool in your ORG
2. Use PR_topic_pool as workflow name in yml files
3. Replace ORG name by yours in all yml files
4. Put PR_topic_pool.yml and PR_topic_cmd.yml in all repositories that need PR topic functions
5. Create 2 secrets PR_TOPIC_ID and PR_TOPIC_TOKEN which can access topic pool repo only(ORG or REPO Settings -> Secrets and variables -> Actions)

### How to use topic?
- Maximum 25 characters in length with a-z, A-Z, 0-9, - and _ for topic name
- Add [TOPIC=topic_name] in git commit message occupies one line (not in GitHub Conversation or comment)
- Use @topic_list in PR comment get PRs list which in same topic
- One topic can only have one PR in the same repository
- Bash script example for topic data usage:
```
git clone -q --depth=1 --single-branch -b ORG/topic-topic_name \
https://github.com/ORG/PR_topic_pool.git topic_pool.tmp

for PROJECT_NAME in $(ls topic_pool.tmp)
do
    unset TP_GH_REPO TP_PR_BASE_BRANCH TP_PR_HEAD_BRANCH TP_PR_HEAD_GIT_LOG TP_PR_HEAD_SHA TP_PR_URL
    source topic_pool.tmp/${PROJECT_NAME}
    export TP_PR_HEAD_SHA_URL="https://github.com/${TP_GH_REPO}/commit/${TP_PR_HEAD_SHA}"
done
```

### FAQ
- Why one topic can only have one PR in the same repository?<br>
  Unlike gerrit is patch based change, GitHub PR is branch based change. All related changes are commited to the same PR branch, need not use topic to combine different changes from one repository.

- Why use commit message define topic, Conversation comments are more flexible?<br>
  Conversation comments can be modify by anyone in anytime, software engineering requires traceability and reproducibility without dependencies. Topic info in Git commit message will help for atomic merge and revert without GitHub.

- How to design topic build?<br>
  Please reference [Create a workflow dispatch event](https://docs.github.com/en/rest/actions/workflows?apiVersion=2022-11-28#create-a-workflow-dispatch-event), use GitHub REST API with PR info from topic for topic build in workflow or script

- How to design topic merge?<br>
  Please reference [Merge a pull request](https://docs.github.com/en/rest/pulls/pulls?apiVersion=2022-11-28#merge-a-pull-request), use GitHub REST API with PR info from topic for topic merge in workflow or script

- How to design topic revert?<br>
  Please reference GraphQL API [revertPullRequest](https://docs.github.com/en/graphql/reference/mutations#revertpullrequest), use GitHub GraphQL API with PR info from topic for topic revert in workflow or script

- What is 70P1C?<br>
  70P1C means TOPIC, tribute to Hacker's [Leet (1337)](https://en.wikipedia.org/wiki/Leet)

- Why do not use or implement by GitHub actions?<br>
  Although there are many excellent actions in the GitHub actions community, for [security reasons](https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions), I still choose to implement it in a flat scripting way. For the end user, all code is transparent, no hidden functionality is nested like GitHub actions.<br>
  I am a faithful practitioner of the [Unix philosophy](https://en.wikipedia.org/wiki/Unix_philosophy#Mike_Gancarz:_The_UNIX_Philosophy).

License
=======

The software described in this paper is offered under the GPL-3.0 License and is available at [LICENSE](/LICENSE)

[![License: CC BY-ND 4.0](https://licensebuttons.net/l/by-nd/4.0/80x15.png)](https://creativecommons.org/licenses/by-nd/4.0/)

This documentation is licensed under a Creative Commons Attribution-NoDerivs 4.0 International License

[![License: CC BY-ND 4.0](https://img.shields.io/badge/License-CC_BY--ND_4.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nd/4.0/)

Attribution: Copyright 2024 General Motors LLC
