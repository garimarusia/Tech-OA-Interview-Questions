Project description
Source note: In an Amazon online assessment for a full-time software role, the AI coding task was a movie review site where users rate movies and write reviews. The candidate had to find and fix several bugs. Two they named were the rate limit and the check for unsuitable words in review comments. The instructions were detailed. The repo was large, with both front-end and back-end code, but every bug was on the back end. The candidate used Node.js. The AI assistant could help them find their way around files and functions, but copying and pasting code was not allowed. As far as they could tell there was no test documentation, and some tests were hidden.
We kept the movie review site and the two bug areas the candidate named. The report does not give the code, the routes, the limit numbers, the word list, the matching rules, the other bugs or the tests. Our version is a back-end-only Python or Java service. We chose the third bug (rounding of the average rating), the limit of 3 reviews per 60 seconds, the word list and the whole-word match rule, the routes, the error messages, the seed data and all the tests. Every test is visible.
Our task is about a 35% match to the report.

MovieDB is a movie review website. Signed-in users write a review for a movie, made of a rating and a comment, and each movie shows its review count and average rating. Several bugs in the backend break the review rules. Find and fix them so that:

Each user can post at most 3 reviews in any 60-second window, counted per user.
A review comment that contains an inappropriate word is rejected. Matching ignores case and looks at whole words only.
A movie's average rating is rounded correctly to one decimal place.
All of the bugs are in the backend. This workspace has the backend API only, with no front end. The routes, the sign-in check and the JSON responses already work. You are done when every visible test passes.

Contract
**Sign-in.** Posting a review needs the header Authorization: Bearer <username>; any non-empty username is a signed-in user.

**Routes.**

POST /api/movies/{movie_id}/reviews with a JSON body such as {"rating": 8, "comment": "Great ending."} saves a review and returns 201 with the saved review.
GET /api/movies/{movie_id}/reviews returns 200 {"reviews": [...]} with that movie's reviews, oldest first.
GET /api/movies/{movie_id} returns 200 {"movie_id": ..., "title": ..., "review_count": ..., "average_rating": ...}.
A review is {"id": <integer>, "movie_id": ..., "username": ..., "rating": ..., "comment": ...}. The comment is saved exactly as sent.

**Checks for a new review, in this order.** The first check that fails decides the response, and nothing is saved.

1. No signed-in user: 401 {"detail": "Authentication credentials were not provided."}.
2. Unknown movie: 404 {"detail": "Movie not found."}. The two GET routes return the same 404 for an unknown movie.
3. The body is not a JSON object: 400 {"detail": "Request body must be a JSON object."}.
4. rating is not a JSON integer from 1 to 10: 400 {"detail": "Rating must be an integer from 1 to 10."}.
5. comment is not a string with at least one non-whitespace character: 400 {"detail": "Comment is required."}.
6. The comment contains an inappropriate word: 400 {"detail": "Comment contains inappropriate language."}.
7. The user is over the rate limit: 429 {"detail": "Too many reviews. Try again later."}.

**Inappropriate words.** The list is data/inappropriate_words.json, a JSON array of lowercase words. A word in a comment is a maximal run of ASCII letters (A to Z, a to z); every other character separates words. A comment is rejected when one of its words, compared without regard to case, is in the list. A listed word inside a longer word does not count: with dumb in the list, That was Dumb! is rejected but dumbbell is allowed.

**Rate limit.** Only saved reviews count. A user's saved review counts while it is less than 60 seconds old. When a user already has 3 counted reviews, on any movies, that user's next review is rejected with 429. One user's reviews never limit another user. Example: after reviews saved at 12:00:00, 12:00:01 and 12:00:02, a review at 12:00:59 is rejected and a review at 12:01:00 is accepted.

**Rating summary.** review_count is the number of saved reviews for the movie. average_rating is the mean rating rounded to one decimal place, with halves rounded up: 8.5 for ratings 9 and 8, and 7.3 for ratings 7, 9 and 6. It is null when the movie has no reviews.

Files you get
data/seed.json: the movies and older reviews loaded each time the app starts. movies entries have movie_id and title; reviews entries have movie_id, username, rating, comment and created_at.
data/inappropriate_words.json: the inappropriate-word list.
The backend code, a few unit tests, and the HTTP tests in fastprep_tests/http/. The note for your language and its README name the code files.
Notes for Python
Python uses Django; the backend is in backend/. A new review is checked and saved by create_review in backend/apps/reviews/services.py, which uses backend/apps/reviews/moderation.py for the word check and backend/apps/reviews/rate_limit.py for the limit. The rating summary is in backend/apps/ratings/summary.py. The README has the full file map and the Python details the tests rely on.