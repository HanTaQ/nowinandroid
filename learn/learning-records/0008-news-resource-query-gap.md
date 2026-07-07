# Learning Record 0008 - NewsResourceQuery Gap

## Context

The learner completed the lesson on `CompositeUserNewsResourceRepository`.

## Current Understanding

- Understands that `distinctUntilChanged()` filters consecutive duplicate `Set<String>` emissions before downstream Flow work.
- Understands that `flatMapLatest` creates a new inner Flow for new upstream values and cancels the previous inner Flow.
- Understands that `UserNewsResource` is more useful for UI than raw `NewsResource` because it includes user-related state.

## Current Gap

The learner is not yet comfortable with how `NewsResourceQuery(filterTopicIds = followedTopics)` turns into the actual database-backed news query.

## Next Teaching Move

Use lesson `0009-news-resource-query-and-repository-flow.html` to make the data path explicit:

`Set<String>` -> `NewsResourceQuery` -> `NewsRepository.getNewsResources` -> `NewsResourceDao.getNewsResources` -> `Flow<List<NewsResource>>` -> `combine(UserData)` -> `Flow<List<UserNewsResource>>`.
