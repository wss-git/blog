---
title: 直播监控弹幕
categories:
  - 抖音
excerpt: ''
description: ''
date: 2024-05-14 09:12:26
---

> 代码来源 https://github.com/saermart/DouyinLiveWebFetcher

## 依赖

```
requests==2.31.0
betterproto==2.0.0b6
websocket-client==1.7.0
```

## 代码

main.py
```py
#!/usr/bin/python
# coding:utf-8

import gzip
import random
import re
import string

import requests
import websocket
from douyin import *


def generateMsToken(length=107):
    """
    产生请求头部cookie中的msToken字段，其实为随机的107位字符
    :param length:字符位数
    :return:msToken
    """
    random_str = ''
    base_str = string.ascii_letters + string.digits + '=_'
    _len = len(base_str) - 1
    for _ in range(length):
        random_str += base_str[random.randint(0, _len)]
    return random_str


def generateTtwid():
    """
    产生请求头部cookie中的ttwid字段，访问抖音网页版直播间首页可以获取到响应cookie中的ttwid
    :return: ttwid
    """
    url = "https://live.douyin.com/"
    headers = {
        "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 "
                      "(KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36",
    }
    try:
        response = requests.get(url, headers=headers)
        response.raise_for_status()
    except Exception as err:
        print("【X】request the live url error: ", err)
    else:
        return response.cookies.get('ttwid')


class DouyinLiveWebFetcher:
    
    def __init__(self, live_id):
        """
        直播间弹幕抓取对象
        :param live_id: 直播间的直播id，打开直播间web首页的链接如：https://live.douyin.com/261378947940，
                        其中的261378947940即是live_id
        """
        self.__ttwid = None
        self.__room_id = None
        self.live_id = live_id
        self.live_url = "https://live.douyin.com/"
        self.user_agent = "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) " \
                          "Chrome/120.0.0.0 Safari/537.36"
    
    def start(self):
        self._connectWebSocket()
    
    def stop(self):
        self.ws.close()
    
    @property
    def ttwid(self):
        """
        产生请求头部cookie中的ttwid字段，访问抖音网页版直播间首页可以获取到响应cookie中的ttwid
        :return: ttwid
        """
        if self.__ttwid:
            return self.__ttwid
        headers = {
            "User-Agent": self.user_agent,
        }
        try:
            response = requests.get(self.live_url, headers=headers)
            response.raise_for_status()
        except Exception as err:
            print("【X】Request the live url error: ", err)
        else:
            self.__ttwid = response.cookies.get('ttwid')
            return self.__ttwid
    
    @property
    def room_id(self):
        """
        根据直播间的地址获取到真正的直播间roomId，有时会有错误，可以重试请求解决
        :return:room_id
        """
        if self.__room_id:
            return self.__room_id
        url = self.live_url + self.live_id
        headers = {
            "User-Agent": self.user_agent,
            "cookie": f"ttwid={self.ttwid}&msToken={generateMsToken()}; __ac_nonce=0123407cc00a9e438deb4",
        }
        try:
            response = requests.get(url, headers=headers)
            response.raise_for_status()
        except Exception as err:
            print("【X】Request the live room url error: ", err)
        else:
            match = re.search(r'roomId\\":\\"(\d+)\\"', response.text)
            if match is None or len(match.groups()) < 1:
                print("【X】No match found for roomId")
            
            self.__room_id = match.group(1)
            
            return self.__room_id
    
    def _connectWebSocket(self):
        """
        连接抖音直播间websocket服务器，请求直播间数据
        """
        wss = f"wss://webcast3-ws-web-lq.douyin.com/webcast/im/push/v2/?" \
              f"app_name=douyin_web&version_code=180800&webcast_sdk_version=1.3.0&update_version_code=1.3.0" \
              f"&compress=gzip" \
              f"&internal_ext=internal_src:dim|wss_push_room_id:{self.room_id}|wss_push_did:{self.room_id}" \
              f"|dim_log_id:202302171547011A160A7BAA76660E13ED|fetch_time:1676620021641|seq:1|wss_info:0-1676" \
              f"620021641-0-0|wrds_kvs:WebcastRoomStatsMessage-1676620020691146024_WebcastRoomRankMessage-167661" \
              f"9972726895075_AudienceGiftSyncData-1676619980834317696_HighlightContainerSyncData-2&cursor=t-1676" \
              f"620021641_r-1_d-1_u-1_h-1" \
              f"&host=https://live.douyin.com&aid=6383&live_id=1" \
              f"&did_rule=3&debug=false&endpoint=live_pc&support_wrds=1&" \
              f"im_path=/webcast/im/fetch/&user_unique_id={self.room_id}&" \
              f"device_platform=web&cookie_enabled=true&screen_width=1440&screen_height=900&browser_language=zh&" \
              f"browser_platform=MacIntel&browser_name=Mozilla&" \
              f"browser_version=5.0%20(Macintosh;%20Intel%20Mac%20OS%20X%2010_15_7)%20AppleWebKit/537.36%20(KHTML,%20" \
              f"like%20Gecko)%20Chrome/110.0.0.0%20Safari/537.36&" \
              f"browser_online=true&tz_name=Asia/Shanghai&identity=audience&" \
              f"room_id={self.room_id}&heartbeatDuration=0&signature=00000000"
        headers = {
            "cookie": f"ttwid={self.ttwid}",
            'user-agent': self.user_agent,
        }
        self.ws = websocket.WebSocketApp(wss,
                                         header=headers,
                                         on_open=self._wsOnOpen,
                                         on_message=self._wsOnMessage,
                                         on_error=self._wsOnError,
                                         on_close=self._wsOnClose)
        try:
            self.ws.run_forever()
        except Exception:
            self.stop()
            raise
    
    def _wsOnOpen(self, ws):
        """
        连接建立成功
        """
        print("WebSocket connected.")
    
    def _wsOnMessage(self, ws, message):
        """
        接收到数据
        :param ws: websocket实例
        :param message: 数据
        """
        
        # 根据proto结构体解析对象
        package = PushFrame().parse(message)
        response = Response().parse(gzip.decompress(package.payload))
        
        # 返回直播间服务器链接存活确认消息，便于持续获取数据
        if response.need_ack:
            ack = PushFrame(log_id=package.log_id,
                            payload_type='ack',
                            payload=response.internal_ext.encode('utf-8')
                            ).SerializeToString()
            ws.send(ack, websocket.ABNF.OPCODE_BINARY)
        
        # 根据消息类别解析消息体
        for msg in response.messages_list:
            method = msg.method
            try:
                {
                    'WebcastChatMessage': self._parseChatMsg,  # 聊天消息
                    'WebcastGiftMessage': self._parseGiftMsg,  # 礼物消息
                    'WebcastLikeMessage': self._parseLikeMsg,  # 点赞消息
                    'WebcastMemberMessage': self._parseMemberMsg,  # 进入直播间消息
                    'WebcastSocialMessage': self._parseSocialMsg,  # 关注消息
                    'WebcastRoomUserSeqMessage': self._parseRoomUserSeqMsg,  # 直播间统计
                    'WebcastFansclubMessage': self._parseFansclubMsg,  # 粉丝团消息
                    'WebcastControlMessage': self._parseControlMsg,  # 直播间状态消息
                    'WebcastEmojiChatMessage': self._parseEmojiChatMsg,  # 聊天表情包消息
                    'WebcastRoomStatsMessage': self._parseRoomStatsMsg,  # 直播间统计信息
                    'WebcastRoomMessage': self._parseRoomMsg,  # 直播间信息
                    'WebcastRoomRankMessage': self._parseRankMsg,  # 直播间排行榜信息
                }.get(method)(msg.payload)
            except Exception:
                pass
    
    def _wsOnError(self, ws, error):
        print("WebSocket error: ", error)
    
    def _wsOnClose(self, **kv):
        print("WebSocket connection closed.")
    
    def _parseChatMsg(self, payload):
        '''聊天消息'''
        message = ChatMessage().parse(payload)
        user_name = message.user.nick_name
        user_id = message.user.id
        content = message.content
        print(f"【聊天msg】[{user_id}]{user_name}: {content}")
    
    def _parseGiftMsg(self, payload):
        '''礼物消息'''
        message = GiftMessage().parse(payload)
        user_name = message.user.nick_name
        gift_name = message.gift.name
        gift_cnt = message.combo_count
        print(f"【礼物msg】{user_name} 送出了 {gift_name}x{gift_cnt}")
    
    def _parseLikeMsg(self, payload):
        '''点赞消息'''
        message = LikeMessage().parse(payload)
        user_name = message.user.nick_name
        count = message.count
        print(f"【点赞msg】{user_name} 点了{count}个赞")
    
    def _parseMemberMsg(self, payload):
        '''进入直播间消息'''
        message = MemberMessage().parse(payload)
        print(f'msg: {message}')
        user_name = message.user.nick_name
        user_id = message.user.id
        gender = ["女", "男"][message.user.gender]
        print(f"【进场msg】[{user_id}][{gender}]{user_name} 进入了直播间")
    
    def _parseSocialMsg(self, payload):
        '''关注消息'''
        message = SocialMessage().parse(payload)
        user_name = message.user.nick_name
        user_id = message.user.id
        print(f"【关注msg】[{user_id}]{user_name} 关注了主播")
    
    def _parseRoomUserSeqMsg(self, payload):
        '''直播间统计'''
        message = RoomUserSeqMessage().parse(payload)
        current = message.total
        total = message.total_pv_for_anchor
        print(f"【统计msg】当前观看人数: {current}, 累计观看人数: {total}")
    
    def _parseFansclubMsg(self, payload):
        '''粉丝团消息'''
        message = FansclubMessage().parse(payload)
        content = message.content
        print(f"【粉丝团msg】 {content}")
    
    def _parseEmojiChatMsg(self, payload):
        '''聊天表情包消息'''
        message = EmojiChatMessage().parse(payload)
        emoji_id = message.emoji_id
        user = message.user
        common = message.common
        default_content = message.default_content
        print(f"【聊天表情包id】 {emoji_id},user：{user},common:{common},default_content:{default_content}")
    
    def _parseRoomMsg(self, payload):
        message = RoomMessage().parse(payload)
        common = message.common
        room_id = common.room_id
        print(f"【直播间msg】直播间id:{room_id}")
    
    def _parseRoomStatsMsg(self, payload):
        message = RoomStatsMessage().parse(payload)
        display_long = message.display_long
        print(f"【直播间统计msg】{display_long}")
    
    def _parseRankMsg(self, payload):
        message = RoomRankMessage().parse(payload)
        ranks_list = message.ranks_list
        print(f"【直播间排行榜msg】{ranks_list}")
    
    def _parseControlMsg(self, payload):
        '''直播间状态消息'''
        message = ControlMessage().parse(payload)
        
        if message.status == 3:
            print("直播间已结束")
            self.stop()
```

douyin.py
```py
# Generated by the protocol buffer compiler.  DO NOT EDIT!
# sources: douyin.proto
# plugin: python-betterproto
from dataclasses import dataclass
from typing import Dict, List

import betterproto


class CommentTypeTag(betterproto.Enum):
    COMMENTTYPETAGUNKNOWN = 0
    COMMENTTYPETAGSTAR = 1


class RoomMsgTypeEnum(betterproto.Enum):
    """
    from https://github.com/scx567888/live-room-watcher/blob/master/src/main/pr
    oto/douyin_hack/webcast/im/RoomMsgTypeEnum.proto
    """

    DEFAULTROOMMSG = 0
    ECOMLIVEREPLAYSAVEROOMMSG = 1
    CONSUMERRELATIONROOMMSG = 2
    JUMANJIDATAAUTHNOTIFYMSG = 3
    VSWELCOMEMSG = 4
    MINORREFUNDMSG = 5
    PAIDLIVEROOMNOTIFYANCHORMSG = 6
    HOSTTEAMSYSTEMMSG = 7


@dataclass
class Response(betterproto.Message):
    messages_list: List["Message"] = betterproto.message_field(1)
    cursor: str = betterproto.string_field(2)
    fetch_interval: int = betterproto.uint64_field(3)
    now: int = betterproto.uint64_field(4)
    internal_ext: str = betterproto.string_field(5)
    fetch_type: int = betterproto.uint32_field(6)
    route_params: Dict[str, str] = betterproto.map_field(
        7, betterproto.TYPE_STRING, betterproto.TYPE_STRING
    )
    heartbeat_duration: int = betterproto.uint64_field(8)
    need_ack: bool = betterproto.bool_field(9)
    push_server: str = betterproto.string_field(10)
    live_cursor: str = betterproto.string_field(11)
    history_no_more: bool = betterproto.bool_field(12)


@dataclass
class Message(betterproto.Message):
    method: str = betterproto.string_field(1)
    payload: bytes = betterproto.bytes_field(2)
    msg_id: int = betterproto.int64_field(3)
    msg_type: int = betterproto.int32_field(4)
    offset: int = betterproto.int64_field(5)
    need_wrds_store: bool = betterproto.bool_field(6)
    wrds_version: int = betterproto.int64_field(7)
    wrds_sub_key: str = betterproto.string_field(8)


@dataclass
class EmojiChatMessage(betterproto.Message):
    common: "Common" = betterproto.message_field(1)
    user: "User" = betterproto.message_field(2)
    emoji_id: int = betterproto.int64_field(3)
    emoji_content: "Text" = betterproto.message_field(4)
    default_content: str = betterproto.string_field(5)
    background_image: "Image" = betterproto.message_field(6)
    from_intercom: bool = betterproto.bool_field(7)
    intercom_hide_user_card: bool = betterproto.bool_field(8)


@dataclass
class ChatMessage(betterproto.Message):
    """聊天"""

    common: "Common" = betterproto.message_field(1)
    user: "User" = betterproto.message_field(2)
    content: str = betterproto.string_field(3)
    visible_to_sender: bool = betterproto.bool_field(4)
    background_image: "Image" = betterproto.message_field(5)
    full_screen_text_color: str = betterproto.string_field(6)
    background_image_v2: "Image" = betterproto.message_field(7)
    public_area_common: "PublicAreaCommon" = betterproto.message_field(9)
    gift_image: "Image" = betterproto.message_field(10)
    agree_msg_id: int = betterproto.uint64_field(11)
    priority_level: int = betterproto.uint32_field(12)
    landscape_area_common: "LandscapeAreaCommon" = betterproto.message_field(13)
    event_time: int = betterproto.uint64_field(15)
    send_review: bool = betterproto.bool_field(16)
    from_intercom: bool = betterproto.bool_field(17)
    intercom_hide_user_card: bool = betterproto.bool_field(18)
    # repeated chatTagsList = 19;
    chat_by: str = betterproto.string_field(20)
    individual_chat_priority: int = betterproto.uint32_field(21)
    rtf_content: "Text" = betterproto.message_field(22)


@dataclass
class LandscapeAreaCommon(betterproto.Message):
    show_head: bool = betterproto.bool_field(1)
    show_nickname: bool = betterproto.bool_field(2)
    show_font_color: bool = betterproto.bool_field(3)
    color_value_list: List[str] = betterproto.string_field(4)
    comment_type_tags_list: List["CommentTypeTag"] = betterproto.enum_field(5)


@dataclass
class RoomUserSeqMessage(betterproto.Message):
    common: "Common" = betterproto.message_field(1)
    ranks_list: List["RoomUserSeqMessageContributor"] = betterproto.message_field(2)
    total: int = betterproto.int64_field(3)
    pop_str: str = betterproto.string_field(4)
    seats_list: List["RoomUserSeqMessageContributor"] = betterproto.message_field(5)
    popularity: int = betterproto.int64_field(6)
    total_user: int = betterproto.int64_field(7)
    total_user_str: str = betterproto.string_field(8)
    total_str: str = betterproto.string_field(9)
    online_user_for_anchor: str = betterproto.string_field(10)
    total_pv_for_anchor: str = betterproto.string_field(11)
    up_right_stats_str: str = betterproto.string_field(12)
    up_right_stats_str_complete: str = betterproto.string_field(13)


@dataclass
class CommonTextMessage(betterproto.Message):
    common: "Common" = betterproto.message_field(1)
    user: "User" = betterproto.message_field(2)
    scene: str = betterproto.string_field(3)


@dataclass
class UpdateFanTicketMessage(betterproto.Message):
    common: "Common" = betterproto.message_field(1)
    room_fan_ticket_count_text: str = betterproto.string_field(2)
    room_fan_ticket_count: int = betterproto.uint64_field(3)
    force_update: bool = betterproto.bool_field(4)


@dataclass
class RoomUserSeqMessageContributor(betterproto.Message):
    score: int = betterproto.uint64_field(1)
    user: "User" = betterproto.message_field(2)
    rank: int = betterproto.uint64_field(3)
    delta: int = betterproto.uint64_field(4)
    is_hidden: bool = betterproto.bool_field(5)
    score_description: str = betterproto.string_field(6)
    exactly_score: str = betterproto.string_field(7)


@dataclass
class GiftMessage(betterproto.Message):
    """礼物消息"""

    common: "Common" = betterproto.message_field(1)
    gift_id: int = betterproto.uint64_field(2)
    fan_ticket_count: int = betterproto.uint64_field(3)
    group_count: int = betterproto.uint64_field(4)
    repeat_count: int = betterproto.uint64_field(5)
    combo_count: int = betterproto.uint64_field(6)
    user: "User" = betterproto.message_field(7)
    to_user: "User" = betterproto.message_field(8)
    repeat_end: int = betterproto.uint32_field(9)
    text_effect: "TextEffect" = betterproto.message_field(10)
    group_id: int = betterproto.uint64_field(11)
    income_taskgifts: int = betterproto.uint64_field(12)
    room_fan_ticket_count: int = betterproto.uint64_field(13)
    priority: "GiftIMPriority" = betterproto.message_field(14)
    gift: "GiftStruct" = betterproto.message_field(15)
    log_id: str = betterproto.string_field(16)
    send_type: int = betterproto.uint64_field(17)
    public_area_common: "PublicAreaCommon" = betterproto.message_field(18)
    tray_display_text: "Text" = betterproto.message_field(19)
    banned_display_effects: int = betterproto.uint64_field(20)
    # GiftTrayInfo trayInfo = 21;  AssetEffectMixInfo assetEffectMixInfo = 22;
    display_for_self: bool = betterproto.bool_field(25)
    interact_gift_info: str = betterproto.string_field(26)
    diy_item_info: str = betterproto.string_field(27)
    min_asset_set_list: List[int] = betterproto.uint64_field(28)
    total_count: int = betterproto.uint64_field(29)
    client_gift_source: int = betterproto.uint32_field(30)
    # AnchorGiftData anchorGift = 31;
    to_user_ids_list: List[int] = betterproto.uint64_field(32)
    send_time: int = betterproto.uint64_field(33)
    force_display_effects: int = betterproto.uint64_field(34)
    trace_id: str = betterproto.string_field(35)
    effect_display_ts: int = betterproto.uint64_field(36)


@dataclass
class GiftStruct(betterproto.Message):
    image: "Image" = betterproto.message_field(1)
    describe: str = betterproto.string_field(2)
    notify: bool = betterproto.bool_field(3)
    duration: int = betterproto.uint64_field(4)
    id: int = betterproto.uint64_field(5)
    # GiftStructFansClubInfo fansclubInfo = 6;
    for_linkmic: bool = betterproto.bool_field(7)
    doodle: bool = betterproto.bool_field(8)
    for_fansclub: bool = betterproto.bool_field(9)
    combo: bool = betterproto.bool_field(10)
    type: int = betterproto.uint32_field(11)
    diamond_count: int = betterproto.uint32_field(12)
    is_displayed_on_panel: bool = betterproto.bool_field(13)
    primary_effect_id: int = betterproto.uint64_field(14)
    gift_label_icon: "Image" = betterproto.message_field(15)
    name: str = betterproto.string_field(16)
    region: str = betterproto.string_field(17)
    manual: str = betterproto.string_field(18)
    for_custom: bool = betterproto.bool_field(19)
    # specialEffectsMap = 20;
    icon: "Image" = betterproto.message_field(21)
    action_type: int = betterproto.uint32_field(22)


@dataclass
class GiftIMPriority(betterproto.Message):
    queue_sizes_list: List[int] = betterproto.uint64_field(1)
    self_queue_priority: int = betterproto.uint64_field(2)
    priority: int = betterproto.uint64_field(3)


@dataclass
class TextEffect(betterproto.Message):
    portrait: "TextEffectDetail" = betterproto.message_field(1)
    landscape: "TextEffectDetail" = betterproto.message_field(2)


@dataclass
class TextEffectDetail(betterproto.Message):
    text: "Text" = betterproto.message_field(1)
    text_font_size: int = betterproto.uint32_field(2)
    background: "Image" = betterproto.message_field(3)
    start: int = betterproto.uint32_field(4)
    duration: int = betterproto.uint32_field(5)
    x: int = betterproto.uint32_field(6)
    y: int = betterproto.uint32_field(7)
    width: int = betterproto.uint32_field(8)
    height: int = betterproto.uint32_field(9)
    shadow_dx: int = betterproto.uint32_field(10)
    shadow_dy: int = betterproto.uint32_field(11)
    shadow_radius: int = betterproto.uint32_field(12)
    shadow_color: str = betterproto.string_field(13)
    stroke_color: str = betterproto.string_field(14)
    stroke_width: int = betterproto.uint32_field(15)


@dataclass
class MemberMessage(betterproto.Message):
    """成员消息"""

    common: "Common" = betterproto.message_field(1)
    user: "User" = betterproto.message_field(2)
    member_count: int = betterproto.uint64_field(3)
    operator: "User" = betterproto.message_field(4)
    is_set_to_admin: bool = betterproto.bool_field(5)
    is_top_user: bool = betterproto.bool_field(6)
    rank_score: int = betterproto.uint64_field(7)
    top_user_no: int = betterproto.uint64_field(8)
    enter_type: int = betterproto.uint64_field(9)
    action: int = betterproto.uint64_field(10)
    action_description: str = betterproto.string_field(11)
    user_id: int = betterproto.uint64_field(12)
    effect_config: "EffectConfig" = betterproto.message_field(13)
    pop_str: str = betterproto.string_field(14)
    enter_effect_config: "EffectConfig" = betterproto.message_field(15)
    background_image: "Image" = betterproto.message_field(16)
    background_image_v2: "Image" = betterproto.message_field(17)
    anchor_display_text: "Text" = betterproto.message_field(18)
    public_area_common: "PublicAreaCommon" = betterproto.message_field(19)
    user_enter_tip_type: int = betterproto.uint64_field(20)
    anchor_enter_tip_type: int = betterproto.uint64_field(21)


@dataclass
class PublicAreaCommon(betterproto.Message):
    user_label: "Image" = betterproto.message_field(1)
    user_consume_in_room: int = betterproto.uint64_field(2)
    user_send_gift_cnt_in_room: int = betterproto.uint64_field(3)


@dataclass
class EffectConfig(betterproto.Message):
    type: int = betterproto.uint64_field(1)
    icon: "Image" = betterproto.message_field(2)
    avatar_pos: int = betterproto.uint64_field(3)
    text: "Text" = betterproto.message_field(4)
    text_icon: "Image" = betterproto.message_field(5)
    stay_time: int = betterproto.uint32_field(6)
    anim_asset_id: int = betterproto.uint64_field(7)
    badge: "Image" = betterproto.message_field(8)
    flex_setting_array_list: List[int] = betterproto.uint64_field(9)
    text_icon_overlay: "Image" = betterproto.message_field(10)
    animated_badge: "Image" = betterproto.message_field(11)
    has_sweep_light: bool = betterproto.bool_field(12)
    text_flex_setting_array_list: List[int] = betterproto.uint64_field(13)
    center_anim_asset_id: int = betterproto.uint64_field(14)
    dynamic_image: "Image" = betterproto.message_field(15)
    extra_map: Dict[str, str] = betterproto.map_field(
        16, betterproto.TYPE_STRING, betterproto.TYPE_STRING
    )
    mp4_anim_asset_id: int = betterproto.uint64_field(17)
    priority: int = betterproto.uint64_field(18)
    max_wait_time: int = betterproto.uint64_field(19)
    dress_id: str = betterproto.string_field(20)
    alignment: int = betterproto.uint64_field(21)
    alignment_offset: int = betterproto.uint64_field(22)


@dataclass
class Text(betterproto.Message):
    key: str = betterproto.string_field(1)
    default_patter: str = betterproto.string_field(2)
    default_format: "TextFormat" = betterproto.message_field(3)
    pieces_list: List["TextPiece"] = betterproto.message_field(4)


@dataclass
class TextPiece(betterproto.Message):
    type: bool = betterproto.bool_field(1)
    format: "TextFormat" = betterproto.message_field(2)
    string_value: str = betterproto.string_field(3)
    user_value: "TextPieceUser" = betterproto.message_field(4)
    gift_value: "TextPieceGift" = betterproto.message_field(5)
    heart_value: "TextPieceHeart" = betterproto.message_field(6)
    pattern_ref_value: "TextPiecePatternRef" = betterproto.message_field(7)
    image_value: "TextPieceImage" = betterproto.message_field(8)


@dataclass
class TextPieceImage(betterproto.Message):
    image: "Image" = betterproto.message_field(1)
    scaling_rate: float = betterproto.float_field(2)


@dataclass
class TextPiecePatternRef(betterproto.Message):
    key: str = betterproto.string_field(1)
    default_pattern: str = betterproto.string_field(2)


@dataclass
class TextPieceHeart(betterproto.Message):
    color: str = betterproto.string_field(1)


@dataclass
class TextPieceGift(betterproto.Message):
    gift_id: int = betterproto.uint64_field(1)
    name_ref: "PatternRef" = betterproto.message_field(2)


@dataclass
class PatternRef(betterproto.Message):
    key: str = betterproto.string_field(1)
    default_pattern: str = betterproto.string_field(2)


@dataclass
class TextPieceUser(betterproto.Message):
    user: "User" = betterproto.message_field(1)
    with_colon: bool = betterproto.bool_field(2)


@dataclass
class TextFormat(betterproto.Message):
    color: str = betterproto.string_field(1)
    bold: bool = betterproto.bool_field(2)
    italic: bool = betterproto.bool_field(3)
    weight: int = betterproto.uint32_field(4)
    italic_angle: int = betterproto.uint32_field(5)
    font_size: int = betterproto.uint32_field(6)
    use_heigh_light_color: bool = betterproto.bool_field(7)
    use_remote_clor: bool = betterproto.bool_field(8)


@dataclass
class LikeMessage(betterproto.Message):
    """点赞"""

    common: "Common" = betterproto.message_field(1)
    count: int = betterproto.uint64_field(2)
    total: int = betterproto.uint64_field(3)
    color: int = betterproto.uint64_field(4)
    user: "User" = betterproto.message_field(5)
    icon: str = betterproto.string_field(6)
    double_like_detail: "DoubleLikeDetail" = betterproto.message_field(7)
    display_control_info: "DisplayControlInfo" = betterproto.message_field(8)
    linkmic_guest_uid: int = betterproto.uint64_field(9)
    scene: str = betterproto.string_field(10)
    pico_display_info: "PicoDisplayInfo" = betterproto.message_field(11)


@dataclass
class SocialMessage(betterproto.Message):
    common: "Common" = betterproto.message_field(1)
    user: "User" = betterproto.message_field(2)
    share_type: int = betterproto.uint64_field(3)
    action: int = betterproto.uint64_field(4)
    share_target: str = betterproto.string_field(5)
    follow_count: int = betterproto.uint64_field(6)
    public_area_common: "PublicAreaCommon" = betterproto.message_field(7)


@dataclass
class PicoDisplayInfo(betterproto.Message):
    combo_sum_count: int = betterproto.uint64_field(1)
    emoji: str = betterproto.string_field(2)
    emoji_icon: "Image" = betterproto.message_field(3)
    emoji_text: str = betterproto.string_field(4)


@dataclass
class DoubleLikeDetail(betterproto.Message):
    double_flag: bool = betterproto.bool_field(1)
    seq_id: int = betterproto.uint32_field(2)
    renewals_num: int = betterproto.uint32_field(3)
    triggers_num: int = betterproto.uint32_field(4)


@dataclass
class DisplayControlInfo(betterproto.Message):
    show_text: bool = betterproto.bool_field(1)
    show_icons: bool = betterproto.bool_field(2)


@dataclass
class EpisodeChatMessage(betterproto.Message):
    common: "Message" = betterproto.message_field(1)
    user: "User" = betterproto.message_field(2)
    content: str = betterproto.string_field(3)
    visible_to_sende: bool = betterproto.bool_field(4)
    # BackgroundImage backgroundImage = 5;   PublicAreaCommon publicAreaCommon =
    # 6;
    gift_image: "Image" = betterproto.message_field(7)
    agree_msg_id: int = betterproto.uint64_field(8)
    color_value_list: List[str] = betterproto.string_field(9)


@dataclass
class MatchAgainstScoreMessage(betterproto.Message):
    common: "Common" = betterproto.message_field(1)
    against: "Against" = betterproto.message_field(2)
    match_status: int = betterproto.uint32_field(3)
    display_status: int = betterproto.uint32_field(4)


@dataclass
class Against(betterproto.Message):
    left_name: str = betterproto.string_field(1)
    left_logo: "Image" = betterproto.message_field(2)
    left_goal: str = betterproto.string_field(3)
    # LeftPlayersList leftPlayersList = 4;  LeftGoalStageDetail
    # leftGoalStageDetail = 5;
    right_name: str = betterproto.string_field(6)
    right_logo: "Image" = betterproto.message_field(7)
    right_goal: str = betterproto.string_field(8)
    # RightPlayersList rightPlayersList  = 9;  RightGoalStageDetail
    # rightGoalStageDetail = 10;
    timestamp: int = betterproto.uint64_field(11)
    version: int = betterproto.uint64_field(12)
    left_team_id: int = betterproto.uint64_field(13)
    right_team_id: int = betterproto.uint64_field(14)
    diff_sei2abs_second: int = betterproto.uint64_field(15)
    final_goal_stage: int = betterproto.uint32_field(16)
    current_goal_stage: int = betterproto.uint32_field(17)
    left_score_addition: int = betterproto.uint32_field(18)
    right_score_addition: int = betterproto.uint32_field(19)
    left_goal_int: int = betterproto.uint64_field(20)
    right_goal_int: int = betterproto.uint64_field(21)


@dataclass
class Common(betterproto.Message):
    method: str = betterproto.string_field(1)
    msg_id: int = betterproto.uint64_field(2)
    room_id: int = betterproto.uint64_field(3)
    create_time: int = betterproto.uint64_field(4)
    monitor: int = betterproto.uint32_field(5)
    is_show_msg: bool = betterproto.bool_field(6)
    describe: str = betterproto.string_field(7)
    # DisplayText displayText = 8;
    fold_type: int = betterproto.uint64_field(9)
    anchor_fold_type: int = betterproto.uint64_field(10)
    priority_score: int = betterproto.uint64_field(11)
    log_id: str = betterproto.string_field(12)
    msg_process_filter_k: str = betterproto.string_field(13)
    msg_process_filter_v: str = betterproto.string_field(14)
    user: "User" = betterproto.message_field(15)
    # Room room = 16;
    anchor_fold_type_v2: int = betterproto.uint64_field(17)
    process_at_sei_time_ms: int = betterproto.uint64_field(18)
    random_dispatch_ms: int = betterproto.uint64_field(19)
    is_dispatch: bool = betterproto.bool_field(20)
    channel_id: int = betterproto.uint64_field(21)
    diff_sei2abs_second: int = betterproto.uint64_field(22)
    anchor_fold_duration: int = betterproto.uint64_field(23)


@dataclass
class User(betterproto.Message):
    id: int = betterproto.uint64_field(1)
    short_id: int = betterproto.uint64_field(2)
    nick_name: str = betterproto.string_field(3)
    gender: int = betterproto.uint32_field(4)
    signature: str = betterproto.string_field(5)
    level: int = betterproto.uint32_field(6)
    birthday: int = betterproto.uint64_field(7)
    telephone: str = betterproto.string_field(8)
    avatar_thumb: "Image" = betterproto.message_field(9)
    avatar_medium: "Image" = betterproto.message_field(10)
    avatar_large: "Image" = betterproto.message_field(11)
    verified: bool = betterproto.bool_field(12)
    experience: int = betterproto.uint32_field(13)
    city: str = betterproto.string_field(14)
    status: int = betterproto.int32_field(15)
    create_time: int = betterproto.uint64_field(16)
    modify_time: int = betterproto.uint64_field(17)
    secret: int = betterproto.uint32_field(18)
    share_qrcode_uri: str = betterproto.string_field(19)
    income_share_percent: int = betterproto.uint32_field(20)
    badge_image_list: List["Image"] = betterproto.message_field(21)
    follow_info: "FollowInfo" = betterproto.message_field(22)
    pay_grade: "PayGrade" = betterproto.message_field(23)
    fans_club: "FansClub" = betterproto.message_field(24)
    # Border Border = 25;
    special_id: str = betterproto.string_field(26)
    avatar_border: "Image" = betterproto.message_field(27)
    medal: "Image" = betterproto.message_field(28)
    real_time_icons_list: List["Image"] = betterproto.message_field(29)
    display_id: str = betterproto.string_field(38)
    sec_uid: str = betterproto.string_field(46)
    fan_ticket_count: int = betterproto.uint64_field(1022)
    id_str: str = betterproto.string_field(1028)
    age_range: int = betterproto.uint32_field(1045)


@dataclass
class PayGrade(betterproto.Message):
    total_diamond_count: int = betterproto.int64_field(1)
    diamond_icon: "Image" = betterproto.message_field(2)
    name: str = betterproto.string_field(3)
    icon: "Image" = betterproto.message_field(4)
    next_name: str = betterproto.string_field(5)
    level: int = betterproto.int64_field(6)
    next_icon: "Image" = betterproto.message_field(7)
    next_diamond: int = betterproto.int64_field(8)
    now_diamond: int = betterproto.int64_field(9)
    this_grade_min_diamond: int = betterproto.int64_field(10)
    this_grade_max_diamond: int = betterproto.int64_field(11)
    pay_diamond_bak: int = betterproto.int64_field(12)
    grade_describe: str = betterproto.string_field(13)
    grade_icon_list: List["GradeIcon"] = betterproto.message_field(14)
    screen_chat_type: int = betterproto.int64_field(15)
    im_icon: "Image" = betterproto.message_field(16)
    im_icon_with_level: "Image" = betterproto.message_field(17)
    live_icon: "Image" = betterproto.message_field(18)
    new_im_icon_with_level: "Image" = betterproto.message_field(19)
    new_live_icon: "Image" = betterproto.message_field(20)
    upgrade_need_consume: int = betterproto.int64_field(21)
    next_privileges: str = betterproto.string_field(22)
    background: "Image" = betterproto.message_field(23)
    background_back: "Image" = betterproto.message_field(24)
    score: int = betterproto.int64_field(25)
    buff_info: "GradeBuffInfo" = betterproto.message_field(26)
    grade_banner: str = betterproto.string_field(1001)
    profile_dialog_bg: "Image" = betterproto.message_field(1002)
    profile_dialog_bg_back: "Image" = betterproto.message_field(1003)


@dataclass
class FansClub(betterproto.Message):
    data: "FansClubData" = betterproto.message_field(1)
    prefer_data: Dict[int, "FansClubData"] = betterproto.map_field(
        2, betterproto.TYPE_INT32, betterproto.TYPE_MESSAGE
    )


@dataclass
class FansClubData(betterproto.Message):
    club_name: str = betterproto.string_field(1)
    level: int = betterproto.int32_field(2)
    user_fans_club_status: int = betterproto.int32_field(3)
    badge: "UserBadge" = betterproto.message_field(4)
    available_gift_ids: List[int] = betterproto.int64_field(5)
    anchor_id: int = betterproto.int64_field(6)


@dataclass
class UserBadge(betterproto.Message):
    icons: Dict[int, "Image"] = betterproto.map_field(
        1, betterproto.TYPE_INT32, betterproto.TYPE_MESSAGE
    )
    title: str = betterproto.string_field(2)


@dataclass
class GradeBuffInfo(betterproto.Message):
    pass


@dataclass
class Border(betterproto.Message):
    pass


@dataclass
class GradeIcon(betterproto.Message):
    icon: "Image" = betterproto.message_field(1)
    icon_diamond: int = betterproto.int64_field(2)
    level: int = betterproto.int64_field(3)
    level_str: str = betterproto.string_field(4)


@dataclass
class FollowInfo(betterproto.Message):
    following_count: int = betterproto.uint64_field(1)
    follower_count: int = betterproto.uint64_field(2)
    follow_status: int = betterproto.uint64_field(3)
    push_status: int = betterproto.uint64_field(4)
    remark_name: str = betterproto.string_field(5)
    follower_count_str: str = betterproto.string_field(6)
    following_count_str: str = betterproto.string_field(7)


@dataclass
class Image(betterproto.Message):
    url_list_list: List[str] = betterproto.string_field(1)
    uri: str = betterproto.string_field(2)
    height: int = betterproto.uint64_field(3)
    width: int = betterproto.uint64_field(4)
    avg_color: str = betterproto.string_field(5)
    image_type: int = betterproto.uint32_field(6)
    open_web_url: str = betterproto.string_field(7)
    content: "ImageContent" = betterproto.message_field(8)
    is_animated: bool = betterproto.bool_field(9)
    flex_setting_list: "NinePatchSetting" = betterproto.message_field(10)
    text_setting_list: "NinePatchSetting" = betterproto.message_field(11)


@dataclass
class NinePatchSetting(betterproto.Message):
    setting_list_list: List[str] = betterproto.string_field(1)


@dataclass
class ImageContent(betterproto.Message):
    name: str = betterproto.string_field(1)
    font_color: str = betterproto.string_field(2)
    level: int = betterproto.uint64_field(3)
    alternative_text: str = betterproto.string_field(4)


@dataclass
class PushFrame(betterproto.Message):
    seq_id: int = betterproto.uint64_field(1)
    log_id: int = betterproto.uint64_field(2)
    service: int = betterproto.uint64_field(3)
    method: int = betterproto.uint64_field(4)
    headers_list: List["HeadersList"] = betterproto.message_field(5)
    payload_encoding: str = betterproto.string_field(6)
    payload_type: str = betterproto.string_field(7)
    payload: bytes = betterproto.bytes_field(8)


@dataclass
class Kk(betterproto.Message):
    k: int = betterproto.uint32_field(14)


@dataclass
class SendMessageBody(betterproto.Message):
    conversation_id: str = betterproto.string_field(1)
    conversation_type: int = betterproto.uint32_field(2)
    conversation_short_id: int = betterproto.uint64_field(3)
    content: str = betterproto.string_field(4)
    ext: List["ExtList"] = betterproto.message_field(5)
    message_type: int = betterproto.uint32_field(6)
    ticket: str = betterproto.string_field(7)
    client_message_id: str = betterproto.string_field(8)


@dataclass
class ExtList(betterproto.Message):
    key: str = betterproto.string_field(1)
    value: str = betterproto.string_field(2)


@dataclass
class Rsp(betterproto.Message):
    a: int = betterproto.int32_field(1)
    b: int = betterproto.int32_field(2)
    c: int = betterproto.int32_field(3)
    d: str = betterproto.string_field(4)
    e: int = betterproto.int32_field(5)
    f: "RspF" = betterproto.message_field(6)
    g: str = betterproto.string_field(7)
    h: int = betterproto.uint64_field(10)
    i: int = betterproto.uint64_field(11)
    j: int = betterproto.uint64_field(13)


@dataclass
class RspF(betterproto.Message):
    q1: int = betterproto.uint64_field(1)
    q3: int = betterproto.uint64_field(3)
    q4: str = betterproto.string_field(4)
    q5: int = betterproto.uint64_field(5)


@dataclass
class PreMessage(betterproto.Message):
    cmd: int = betterproto.uint32_field(1)
    sequence_id: int = betterproto.uint32_field(2)
    sdk_version: str = betterproto.string_field(3)
    token: str = betterproto.string_field(4)
    refer: int = betterproto.uint32_field(5)
    inbox_type: int = betterproto.uint32_field(6)
    build_number: str = betterproto.string_field(7)
    send_message_body: "SendMessageBody" = betterproto.message_field(8)
    # 字段名待定
    aa: str = betterproto.string_field(9)
    device_platform: str = betterproto.string_field(11)
    headers: List["HeadersList"] = betterproto.message_field(15)
    auth_type: int = betterproto.uint32_field(18)
    biz: str = betterproto.string_field(21)
    access: str = betterproto.string_field(22)


@dataclass
class HeadersList(betterproto.Message):
    key: str = betterproto.string_field(1)
    value: str = betterproto.string_field(2)


@dataclass
class LiveShoppingMessage(betterproto.Message):
    common: "Common" = betterproto.message_field(1)
    msg_type: int = betterproto.int32_field(2)
    promotion_id: int = betterproto.int64_field(4)


@dataclass
class RoomStatsMessage(betterproto.Message):
    common: "Common" = betterproto.message_field(1)
    display_short: str = betterproto.string_field(2)
    display_middle: str = betterproto.string_field(3)
    display_long: str = betterproto.string_field(4)
    display_value: int = betterproto.int64_field(5)
    display_version: int = betterproto.int64_field(6)
    incremental: bool = betterproto.bool_field(7)
    is_hidden: bool = betterproto.bool_field(8)
    total: int = betterproto.int64_field(9)
    display_type: int = betterproto.int64_field(10)


@dataclass
class ProductInfo(betterproto.Message):
    promotion_id: int = betterproto.int64_field(1)
    index: int = betterproto.int32_field(2)
    target_flash_uids_list: List[int] = betterproto.int64_field(3)
    explain_type: int = betterproto.int64_field(4)


@dataclass
class CategoryInfo(betterproto.Message):
    id: int = betterproto.int32_field(1)
    name: str = betterproto.string_field(2)
    promotion_ids_list: List[int] = betterproto.int64_field(3)
    type: str = betterproto.string_field(4)
    unique_index: str = betterproto.string_field(5)


@dataclass
class ProductChangeMessage(betterproto.Message):
    common: "Common" = betterproto.message_field(1)
    update_timestamp: int = betterproto.int64_field(2)
    update_toast: str = betterproto.string_field(3)
    update_product_info_list: List["ProductInfo"] = betterproto.message_field(4)
    total: int = betterproto.int64_field(5)
    update_category_info_list: List["CategoryInfo"] = betterproto.message_field(8)


@dataclass
class ControlMessage(betterproto.Message):
    """
    from https://github.com/HaoDong108/DouyinBarrageGrab/blob/main/BarrageGrab/
    proto/message.proto status = 3 下播
    """

    common: "Common" = betterproto.message_field(1)
    status: int = betterproto.int32_field(2)


@dataclass
class FansclubMessage(betterproto.Message):
    """
    from https://github.com/HaoDong108/DouyinBarrageGrab/blob/main/BarrageGrab/
    proto/message.proto
    """

    common_info: "Common" = betterproto.message_field(1)
    # 升级是1，加入是2
    type: int = betterproto.int32_field(2)
    content: str = betterproto.string_field(3)
    user: "User" = betterproto.message_field(4)


@dataclass
class RoomRankMessage(betterproto.Message):
    """
    from https://github.com/scx567888/live-room-watcher/blob/master/src/main/pr
    oto/douyin_hack/webcast/im/RoomRankMessage.proto 直播间排行榜
    """

    common: "Common" = betterproto.message_field(1)
    ranks_list: List["RoomRankMessageRoomRank"] = betterproto.message_field(2)


@dataclass
class RoomRankMessageRoomRank(betterproto.Message):
    user: "User" = betterproto.message_field(1)
    score_str: str = betterproto.string_field(2)
    profile_hidden: bool = betterproto.bool_field(3)


@dataclass
class RoomMessage(betterproto.Message):
    """
    from https://github.com/scx567888/live-room-
    watcher/blob/master/src/main/proto/douyin_hack/webcast/im/RoomMessage.proto
    """

    common: "Common" = betterproto.message_field(1)
    content: str = betterproto.string_field(2)
    supprot_landscape: bool = betterproto.bool_field(3)
    roommessagetype: "RoomMsgTypeEnum" = betterproto.enum_field(4)
    system_top_msg: bool = betterproto.bool_field(5)
    forced_guarantee: bool = betterproto.bool_field(6)
    biz_scene: str = betterproto.string_field(20)
    buried_point_map: Dict[str, str] = betterproto.map_field(
        30, betterproto.TYPE_STRING, betterproto.TYPE_STRING
    )
```
