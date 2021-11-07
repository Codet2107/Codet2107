- 👋 Hi, I’m @Codet2107
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...

<!---
Codet2107/Codet2107 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
from InstagramAPI import InstagramAPI
from random import randint
from time import sleep
import requests
import json


class Colour:
    Green, Red, White, Yellow = '\033[92m', '\033[91m', '\033[0m', '\033[93m'


print(Colour.Yellow + '\nINSTASPAM MULTI MACHINE OF DOOM\n' + Colour.White)

insta_accounts = {
    'ACCOUNT1': 'XXXX',
    'ACCOUNT2': 'XXXX'
}

hashtags = {
    'ACCOUNT1': 'HASHTAG',
    'ACCOUNT2': 'HASHTAG'
}


whitelist = [4333514903, 4333514901]
loud = 0


def get_recent_media(instauser):
    url = 'https://www.instapi.io/u/{}'.format(instauser)
    go = False
    while go == False:
        try:
            r = requests.get(url)
            data = r.json()
            medias = []
            for media in data['graphql']['user']['edge_owner_to_timeline_media']['edges']:
                medias.append(media['node']['id'])
            go = True
        except Exception as e:
            print(e)
    return medias


def getIds(api, user_id, friends):
    gotcha = []
    next_max_id = True
    while next_max_id:
        if next_max_id is True:
            next_max_id = ''
        if friends:
            api.getUserFollowings(user_id, maxid=next_max_id)
        else:
            api.getUserFollowers(user_id, maxid=next_max_id)
        gotcha.extend(api.LastJson.get('users', []))
        next_max_id = api.LastJson.get('next_max_id', '')
    return gotcha


def snooze():
    if randint(0, 8) < 5:
        timer = randint(6, 45)
        if loud:
            print('Sleeping for {} seconds'.format(timer))
        sleep(timer)
    else:
        timer = randint(3, 900)
        if loud:
            print('Sleeping for {} seconds'.format(timer))
        sleep(timer)


def friend_check(api, user):
    api.userFriendship(user)
    if api.LastJson["following"]:
        return True
    else:
        return False


def like_tag_feed(api, tag):
    print('Liking media with hashtag {}'.format(tag))
    next_max = 1
    next_max_id = ''
    likes = 0
    for n in range(next_max):
        api.getHashtagFeed(tag, next_max_id)
        temp = api.LastJson
        for post in temp["items"]:
            if post["has_liked"] == False:
                print('Liking {} '.format(post["pk"]))
                api.like(post["pk"])
                likes += 1
                if likes > max_likes:
                    break
                sleep(randint(3, 12))
        try:
            next_max_id = temp["next_max_id"]
        except Exception:
            print("error next_max. Tag: ", tag)
        if likes > max_likes:
            break


def do_follows(api):
    print('Balancing followers/following')
    user_id = api.username_id
    username = api.username
    followers = getIds(api, user_id, False)
    following = getIds(api, user_id, True)
    followers_list = []
    following_list = []
    for follower in followers:
        followers_list.append(follower['pk'])
    for follower in following:
        following_list.append(follower['pk'])
    losers = set(following_list) - set(followers_list)
    lost = len(losers)
    folls = len(followers)
    follg = len(following)

    print('Followers: {}'.format(folls))
    print('Following: {}'.format(follg))
    print('Losers: {}'.format(lost))

    count = 0
    for winner in followers_list:
        if friend_check(api, winner):
            print('Already following {}'.format(winner))
        else:
            api.follow(winner)
            print('Following {}'.format(winner))
        if count > max_followback:
            break
        count += 1
        sleep(randint(3, 15))

    count = 0
    for loser in losers:
        if loser in whitelist:
            print('Whitelisted {}'.format(loser))
        else:
            api.unfollow(loser)
            print('Unfollowed {}'.format(loser))
            if count > max_unfollow:
                break
            count += 1
        sleep(randint(3, 15))


def follow_media_likers(api, instauser):
    print('Checking recent media likers')
    count = 0
    for media in get_recent_media(instauser):
        api.getMediaLikers(mediaId=media)
        info = api.LastJson
        for user in info['users']:
            id = user['pk']
            if friend_check(api, id):
                print('Already following {}'.format(id))
            else:
                api.follow(id)
                print('Following {}'.format(id))
            sleep(randint(3, 15))
        if count > max_media_likers:
            break
        count += 1


for instauser, instapass in insta_accounts.items():
    api = InstagramAPI(instauser, instapass)
    api.login()
    print('Account: {}'.format(api.username))
    #max_likes = randint(5, 25)
    #max_media_likers = randint(2, 5)
    #max_followback = randint(1, 25)
    #max_unfollow = randint(1, 25)
    max_likes = 0
    max_media_likers = 3
    max_followback = 1000
    max_unfollow = 100
    hashtag = hashtags[instauser]
    #like_tag_feed(api, hashtag)
    do_follows(api)
    follow_media_likers(api, instauser)
    api.logout()
