​	

```re
import axios from '@/libs/netAxios'
import { is_Env_TEST_Log } from '@/libs/util.js'
import URLS from '_conf/url'
const signalR = require('@microsoft/signalr')
const NET_URL = URLS.NET_URL

class SignalR {
  constructor(store) {
    this.connection = ''
    this.reConnectionNumber = 0
    this.store = store
  }
  init() {
    const token = sessionStorage.getItem('Token')
    this.connection = new signalR.HubConnectionBuilder()
      .withUrl(`${NET_URL}/merchantHub`, {
        accessTokenFactory: () => token
      })
      .build()
    this.onclose()
    return this.connection
  }
  async start() {
    try {
      this.connection.on(
        'onOrderRefreshAfterPermissionUpdate',
        async (data) => {
          if (data.NeedsOrderRefreshAfterPermissionUpdate) {
            const { code, data } = await axios.request({
              url: '/api/merchant/management/boundMerchants',
              method: 'get',
              params: {
                IsCompanyGroup: true
              }
            })
            if (code * 1 === 20000) {
              if (Array.isArray(data) && data.length) {
                this.store?.commit('setWebOrderList', data)
              }
            }
          }
        }
      )
      this.connection.on('onReceiveIssueMessage', (data) => {
        if (!this.store) return
        this.store?.commit('setUpdateKocPostListStatus', true)
        const deduplication = this.store.getters.getImList.some(
          (item) => item.id === data?.messageEntityId
        )
        if (
          this.store.getters.getImShowGroupId &&
          this.store.getters.getImShowGroupId === data?.groupEntityId &&
          !deduplication
        ) {
          this.store?.commit('setImList', [
            ...this.store.getters.getImList,
            {
              content: data?.content,
              customProperties: data?.customProperties,
              groupId: data?.groupEntityId,
              id: data?.messageEntityId,
              isRevoked: data?.isRevoked,
              isSystem: data?.sendByUserId !== sessionStorage.getItem('userid'),
              payload: null,
              sentBy: data?.sendByUserId,
              sentTime: new Date(),
              type: data?.type,
              url: null
            }
          ])
        }
      })

      this.connection.on('onMerchKocCampaignSettingChange', (data) => {
        this.store?.commit('setKocMenuStatus', data?.isEnabled)
      })

      this.connection.on('onGiveFoodSettingStatusChanged', () => {
        // 處理狀態提示
        // Message.success('订单赠品活動已完成！')
      })

      this.connection
        .start()
        .then(() => {
          this.store?.commit('setConnetStatus', true)
          this.reConnectionNumber = 0
          this.connection.invoke(
            'AddToGroup',
            sessionStorage.getItem('merchId') ?? ''
          )
        })
        .catch((err) => {
          this.store?.commit('setConnetStatus', false)
          is_Env_TEST_Log(err)
        })
    } catch (err) {
      if (this.reConnectionNumber > 5) {
        throw new Error('Network error')
      }
      setTimeout(() => {
        this.reConnectionNumber++
        this.start()
      }, 120000)
      is_Env_TEST_Log(err)
    }
  }
  onclose() {
    this.connection.onclose(async () => {
      this.start()
    })
  }
  invoke(event, params) {
    this.connection.invoke(event, params)
  }
  on(event, callback) {
    this.connection.on(event, callback)
  }
}

export default SignalR
```

