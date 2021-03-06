https://github.com/GolosChain/golos/issues/541

1. Run daemon with enabled `tags`, `social_network` and `follow`
2. Wait HF 18
3. Run and prepare `cli_wallet`
```
set_password 1qaz
unlock 1qaz
import_key 5JVFFWRLwz6JoP9kguuRFfytToGU6cLgBVTL9t6NB3D3BQLbUBS
```
4. Create accounts alice, bob, dave, greg
```
create_account cyberfounder alice "{}" "300.000 GOLOS" true
create_account cyberfounder bob "{}" "300.000 GOLOS" true
create_account cyberfounder dave "{}" "300.000 GOLOS" true
create_account cyberfounder greg "{}" "300.000 GOLOS" true
```
5. Create feed for alice
```
begin_builder_transaction
add_operation_to_builder_transaction 0 ["custom_json", {"required_posting_auths":["alice"],"id": "follow", "json":"[\"follow\", {\"follower\":\"alice\",\"following\":\"bob\",\"what\":[\"blog\"]}]"}]
sign_builder_transaction 0 true
```
6. Create posts (with tags and without tags) and comments. `tags` plugin uses `comment_api_object` to parse tags
```
post_comment alice test "" ptest "Test change1" "<h1>changed 1!</h1><hr>test edit" "{}" true
post_comment bob test "" ptest "Test change1" "<h1>changed 1!</h1><hr>test edit" "{\"tags\":[\"tag1\",\"tag2\"],\"language\":\"RU\"}" true
post_comment dave test alice test "Test change1" "<h1>changed 1!</h1><hr>test edit" "" true
post_comment greg test bob test "Test change1" "<h1>changed 1!</h1><hr>test edit" "" true
```
7. Vote posts
```
vote dave alice test 100 true
vote greg bob test 100 true
```
8. Send requests to `social_network` and validate results
```
curl --data '{"method":"call","params":["social_network", "get_content", ["alice","test"]],"jsonrpc":"2.0","id":0}' http://127.0.0.1:8090/rpc | python -mjson.tool
curl --data '{"method":"call","params":["social_network", "get_content_replies", ["alice","test"]],"jsonrpc":"2.0","id":0}' http://127.0.0.1:8090/rpc | python -mjson.tool
curl --data '{"method":"call","params":["social_network", "get_all_content_replies", ["bob","test"]],"jsonrpc":"2.0","id":0}' http://127.0.0.1:8090/rpc | python -mjson.tool
curl --data '{"method":"call","params":["social_network", "get_replies_by_last_update", ["bob","test",100]],"jsonrpc":"2.0","id":0}' http://127.0.0.1:8090/rpc | python -mjson.tool
```
9. Send requests to `tags` and validate results
```
curl --data '{"method":"call","params":["tags", "get_discussions_by_payout", [{}]],"jsonrpc":"2.0","id":0}' http://127.0.0.1:8090/rpc | python -mjson.tool
curl --data '{"method":"call","params":["tags", "get_discussions_by_trending", [{"select_tags":["tag1"]}]],"jsonrpc":"2.0","id":0}' http://127.0.0.1:8090/rpc | python -mjson.tool
curl --data '{"method":"call","params":["tags", "get_discussions_by_created", [{"filter_tags":["tag1"]}]],"jsonrpc":"2.0","id":0}' http://127.0.0.1:8090/rpc | python -mjson.tool
curl --data '{"method":"call","params":["tags", "get_discussions_by_active", [{"select_authors":["alice"]}]],"jsonrpc":"2.0","id":0}' http://127.0.0.1:8090/rpc | python -mjson.tool
curl --data '{"method":"call","params":["tags", "get_discussions_by_cashout", [{}]],"jsonrpc":"2.0","id":0}' http://127.0.0.1:8090/rpc | python -mjson.tool
curl --data '{"method":"call","params":["tags", "get_discussions_by_votes", [{}]],"jsonrpc":"2.0","id":0}' http://127.0.0.1:8090/rpc | python -mjson.tool
curl --data '{"method":"call","params":["tags", "get_discussions_by_children", [{}]],"jsonrpc":"2.0","id":0}' http://127.0.0.1:8090/rpc | python -mjson.tool
curl --data '{"method":"call","params":["tags", "get_discussions_by_hot", [{}]],"jsonrpc":"2.0","id":0}' http://127.0.0.1:8090/rpc | python -mjson.tool
curl --data '{"method":"call","params":["tags", "get_discussions_by_feed", [{"select_authors":["alice"]}]],"jsonrpc":"2.0","id":0}' http://127.0.0.1:8090/rpc | python -mjson.tool
curl --data '{"method":"call","params":["tags", "get_discussions_by_blog", [{"select_authors":["alice"]}]],"jsonrpc":"2.0","id":0}' http://127.0.0.1:8090/rpc | python -mjson.tool
curl --data '{"method":"call","params":["tags", "get_discussions_by_comments", [{"start_author":"dave"}]],"jsonrpc":"2.0","id":0}' http://127.0.0.1:8090/rpc | python -mjson.tool
curl --data '{"method":"call","params":["tags", "get_discussions_by_promoted", [{}]],"jsonrpc":"2.0","id":0}' http://127.0.0.1:8090/rpc | python -mjson.tool
curl --data '{"method":"call","params":["tags", "get_discussions_by_author_before_date", ["alice","","1970-01-01T00:00:00",10]],"jsonrpc":"2.0","id":0}' http://127.0.0.1:8090/rpc | python -mjson.tool
```
10. Send requests to `follow` and validate results
```
curl --data '{"method":"call","params":["follow", "get_feed_entries",["alice", 0, 10]],"jsonrpc":"2.0","id":0}' http://127.0.0.1:8090/rpc | python -mjson.tool
curl --data '{"method":"call","params":["follow", "get_feed",["alice", 0, 10]],"jsonrpc":"2.0","id":0}' http://127.0.0.1:8090/rpc | python -mjson.tool
curl --data '{"method":"call","params":["follow", "get_blog_entries",["alice", 0, 10]],"jsonrpc":"2.0","id":0}' http://127.0.0.1:8090/rpc | python -mjson.tool
curl --data '{"method":"call","params":["follow", "get_blog",["alice", 0, 10]],"jsonrpc":"2.0","id":0}' http://127.0.0.1:8090/rpc | python -mjson.tool
```
