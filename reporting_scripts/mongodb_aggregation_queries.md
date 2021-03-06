# About
Mongo Aggregation Queries used on the mongo shell to output new collections with results of the aggregation queries

### 1 - Seek Video data
    db.tracking_atoc185x.aggregate([{$match : {$and : [{"event_type" : "seek_video"},{ "parent_data": { $exists: true } }]}}, {$sort : {"parent_data.chapter_display_name" : 1, "parent_data.sequential_display_name" : 1, "parent_data.vertical_display_name" : 1}}, {$out : "seek_video"}], {allowDiskUse : true})

### 2 - Speed change data
    db.tracking_atoc185x.aggregate([{$match : {$and : [{"event_type" : "speed_change_video"},{ "parent_data": { $exists: true } }]}}, {$sort : {"parent_data.chapter_display_name" : 1, "parent_data.sequential_display_name" : 1, "parent_data.vertical_display_name" : 1}}, {$out : "speed_change_video_data"}, {allowDiskUse : true}])

### 3 - Number of video watchers per week
    db.tracking_atoc185x.aggregate([{$group : { _id : { "chapter_name" : "$parent_data.chapter_display_name" ,"sequential_name" : "$parent_data.sequential_display_name","vertical_name" : "$parent_data.vertical_display_name"},students : {$addToSet:"$username"}}}, {$unwind : "$students"} ,{$group : {_id: "$_id", num_of_students : {$sum : 1}}}, {$out : "username_count_chapter_name"}])

### 4 - Number of attempts to get correct answer to a problem
    db.tracking_atoc185x.aggregate([{$match : {'event_type': 'problem_check', 'event_source' : 'server', "event.success" : 'correct'}}, {$group : { _id :  {"username" : "$username", "problem_id" : "$event.problem_id" }, num_of_attempts : {$sum : 1}}}, {$out : 'count_of_correct_attemps'}]) 

### 5 - Number of incorrect attempts to a problem
    db.tracking_atoc185x.aggregate([{$match : {'event_type': 'problem_check', 'event_source' : 'server', "event.success" : 'incorrect'}}, {$group : { _id :  {"username" : "$username", "problem_id" : "$event.problem_id" }, num_of_attempts : {$sum : 1}}}, {$out : 'count_of_incorrect_attemps'}])

### 6 - Number of attempts to a problem by getting the maximum number of attempts by a user on a problem_id
    db.tracking_atoc185x.aggregate([{$match : {event_type : 'problem_check', event_source : 'server'}}, {$group : {_id : {"username" : "$username", "problem_id" : "$event.problem_id"}, max_num_of_attempts : {$max : '$event.attempts'}}}, {$out : "num_of_attempts"}])

### 7 - Average number of attempts for each problem id sorted in descending order (collection num_of_attempts was created from query 6 above)
    db.num_of_attempts.aggregate([{$group : {_id : {"chapter_name" : "$_id.chapter_name","sequential_name" : "$_id.sequential_name", "vertical_name" : "$_id.vertical_name" ,"problem_id" : "$_id.problem_id"}, avg_num_of_attempts : {$avg : "$max_num_of_attempts"}}}, {$sort : {avg_num_of_attempts : -1}}, {$out : "avg_num_of_attempts"}])

### 8 - Count the number of unique users who answered a problem successfully
    db.tracking_atoc185x.aggregate([{$match : {"event_type" : "problem_check", "event_source" : "server", "event.success": correct}}, {$group : { _id : { "chapter_name" : "$parent_data.chapter_display_name" ,"sequential_name" : "$parent_data.sequential_display_name","vertical_name" : "$parent_data.vertical_display_name", "problem_id" : "$event.problem_id"},students : {$addToSet:"$username"}}}, {$unwind : "$students"} ,{$group : {_id: "$_id", num_of_students : {$sum : 1}}}, {$out : {username_unique_count_success_correct}}])

### 9 - Student who never accesses the courses (either one event (they registered) or one event (they unregistered) or two events (they registered and then unregistered)) , (this task was divided into two queries - if it can be done more efficiently, please let us know)
    db.tracking_atoc185x.aggregate([{$group : {_id : {"username" : "$username"}, event : {$addToSet : "$event_type"}}}, {$out : 'students_never_accessed'}], {allowDiskUse : true})

    db.students_never_accessed.aggregate([{$match : {$or : [{$and : [{event : {$size : 1}}, {event : {$in : ["edx.course.enrollment.activated","edx.course.enrollment.deactivated"]}}]},{$and : [{event : {$size : 2}}, {event : {$all : ["edx.course.enrollment.activated","edx.course.enrollment.deactivated"]}}]} ]}}, {$out : "students_never_accessed_number"}])

### 10 - Attempts by a student per problem id
    db.tracking_atoc185x.aggregate([{$match : {event_type : 'problem_check', 'event_source': 'server'}}, {$group : {_id : {"username" : "$username", "problem_id" : "$event.problem_id"}, attempts : {$push : "$event.success"}}}, {$out : "user_attempts_per_problem_id"}])
    
### 11 - Number of unique users in tracking logs
    db.tracking_atoc185x.aggregate([{$group:{"_id":"$username","event_count":{$sum:1}}},{$out:"unregistration_user_count"}])

### 12 - Number of unique users who did something else than registering/unregistering
    db.tracking_atoc185x.aggregate([{$match:{"event_type":{$not:/edx\.course\.enrollment.*/}}},{$group:{"_id":"$username","event_count":{$sum:1}}},{$out:"number_did_something"}])

### 13 - Last event of every unique user
    db.tracking_atoc185x.aggregate([{ $sort: { "time": 1 } }, { $group: { "_id":"$username", "date":{ $last:"$time" }, "last_event_type": { $last:"$event_type" }, "metadata": { $last:"$metadata" }, "parent_data": { $last:"$parent_data" } } }, {$out:"last_event_by_user"} ])

### 14 - Number of tracking events per user
    db.tracking.aggregate([{$group : { _id : "$username", count : {$sum : 1}}},{$out : "tracking_count_per_user"}])

### 15 - Number of accesses to Course Info Page per user
    db.tracking.aggregate([{$match : {event_type : "/courses/McGillX/CHEM181X/1T2014/info"}},{$group : {_id :     "$username", count : {"$sum" : 1}}},{"$out" : "course_info_access_count_per_user"}])

### 16 User Gender
    db.auth_userprofile.aggregate([{$group: {_id : {"user_id" : "$user_id", "gender" : "$gender"}}}, {$out: "user_gender"}])

### 17 User Grades
    db.certificates_generatedcertificate.aggregate([{$group: {_id : {"user_id" : "$user_id", "grade" : "$grade"}}}, {$out: "user_grade"}])

### 18 User Certificate Type
    db.student_courseenrollment.aggregate([{$group: {_id : {"user_id" : "$user_id", "mode" : "$mode"}}}, {$out: "user_certificate_type"}])

### 19 User Year of Birth
    db.auth_userprofile.aggregate([{$group: {_id : {"user_id" : "$user_id", "year_of_birth" : "$year_of_birth"}}}, {$out: "user_year_of_birth"}])

### 20 Level of Education
    db.auth_userprofile.aggregate([{$group: {_id : {"user_id" : "$user_id", "level_of_education" : "$level_of_education"}}}, {$out: "user_level_of_education"}])

### 21 User Country
    db.auth_userprofile.aggregate([{$group: {_id : {"user_id" : "$user_id", "country" : "$country"}}}, {$out: "user_country"}])
