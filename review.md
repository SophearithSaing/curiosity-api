# Review Feature Requirement
- A review endpoint should be added for user to be able to review an active session
- User should be able to review from level 0 until current level, exclusive. If current level is 30, review until 29.
- Live questions that are saved should now have a sessionId attached to be used for review.
- Review question should return a set instead of individual questions.
- Question set to be reviewed should be queried based on newly attached questionId.
