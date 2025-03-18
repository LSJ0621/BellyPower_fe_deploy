<template>
    <v-container>
      <v-row v-if="influencers.length > 0">
        <v-col
          v-for="(influencer, index) in influencers"
          :key="influencer.userId || index"
          cols="12"
          sm="6"
          md="4"
          lg="3"
        >
          <v-card
            @click="goToInfluencerDetail(influencer.nickname)"
            class="influencer-card"
            elevation="1"
          >
            <v-img
              :src="influencer.profileImage || '/defaultProfile.png'"
              height="220"
              cover
            >
              <template v-slot:placeholder>
                <v-row class="fill-height ma-0" align="center" justify="center">
                  <v-progress-circular indeterminate color="primary" />
                </v-row>
              </template>
            </v-img>
  
            <v-card-text class="pa-4">
              <h3 class="nickname text-truncate">{{ influencer.nickname }}</h3>
              <div class="followers-count">
                <v-icon size="18" color="#FF5722">mdi-account-multiple</v-icon>
                <span>{{ influencer.followersCount }} 팔로워</span>
              </div>
            </v-card-text>
          </v-card>
        </v-col>
      </v-row>
  
      <div v-else class="text-center mt-10">
        <v-progress-circular indeterminate color="primary" />
      </div>
    </v-container>
  </template>
  
  <script>
  import axios from 'axios';
  
  export default {
    data() {
      return {
        influencers: [],
        limit: 12,
        isLoading: false
      };
    },
    created() {
      this.fetchInfluencers();
    },
    methods: {
      async fetchInfluencers() {
        this.isLoading = true;
        try {
          const response = await axios.get(
            `${process.env.VUE_APP_API_BASE_URL}/user/top-influencers`,
            { params: { limit: this.limit } }
          );
          this.influencers = response.data || [];
        } catch (error) {
          console.error('❌ 인플루언서 불러오기 실패:', error);
        } finally {
          this.isLoading = false;
        }
      },
      goToInfluencerDetail(nickname) {
        // 유저 닉네임으로 쿼리 파라미터 전달하여 이동
        window.location.href = `${process.env.VUE_APP_API_BASE_URL}/user/yourpage?nickName=${encodeURIComponent(nickname)}`;
      }
    }
  };
  </script>
  
  <style scoped>
  .influencer-card {
    border-radius: 16px;
    overflow: hidden;
    transition: all 0.4s ease;
    cursor: pointer;
  }
  
  .influencer-card:hover {
    transform: translateY(-6px);
    box-shadow: 0 8px 20px rgba(0, 0, 0, 0.12);
  }
  
  .nickname {
    font-size: 1.1rem;
    font-weight: 600;
    margin-bottom: 8px;
    color: #1a1a1a;
  }
  
  .followers-count {
    display: flex;
    align-items: center;
    gap: 6px;
    font-size: 0.9rem;
    color: #FF5722;
  }
  </style>
  