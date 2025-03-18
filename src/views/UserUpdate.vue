<template>
  <v-container fluid class="update-container pa-0">
    <!-- 헤더 섹션 -->
    <div class="header-section mb-8">
      <div class="header-content text-center pa-8">
        <h1 class="text-h4 font-weight-bold mb-2">프로필 수정</h1>
        <p class="text-subtitle-1 text-white">회원님의 정보를 수정할 수 있습니다</p>
      </div>
    </div>

    <!-- 폼 섹션 -->
    <v-container>
      <v-row justify="center">
        <v-col cols="12" md="8" lg="6">
          <v-card class="form-card pa-6" elevation="0">
            <v-form ref="form" @submit.prevent="updateUser">
              <!-- 프로필 사진 섹션 -->
              <div class="profile-section text-center mb-8">
                <div class="profile-upload-container">
                  <input
                    type="file"
                    ref="fileInput"
                    accept="image/*"
                    class="hidden-input"
                    @change="handleFileChange"
                  />
                  <v-avatar
                    size="120"
                    class="profile-avatar"
                    @click="$refs.fileInput.click()"
                  >
                    <v-img
                      :src="previewImage"
                      cover
                    >
                      <template v-slot:placeholder>
                        <v-avatar
                          size="120"
                          color="grey-lighten-2"
                        >
                          <v-icon
                            size="40"
                            color="grey-lighten-1"
                          >
                            mdi-account
                          </v-icon>
                        </v-avatar>
                      </template>
                    </v-img>
                    <div class="upload-overlay">
                      <v-icon color="white" size="24">mdi-camera</v-icon>
                    </div>
                  </v-avatar>
                  <p class="text-caption mt-2 text-grey">클릭하여 프로필 사진 변경</p>
                </div>
              </div>

              <!-- 개인 정보 섹션 -->
              <div class="form-section">
                <h3 class="text-h6 font-weight-medium mb-4">개인 정보</h3>
                <v-text-field
                  v-model="userInfo.userName"
                  label="이름"
                  variant="outlined"
                  density="comfortable"
                  required
                  class="mb-3"
                ></v-text-field>

                <v-text-field
                  v-model="userInfo.userNickName"
                  label="닉네임"
                  variant="outlined"
                  density="comfortable"
                  required
                  class="mb-3"
                ></v-text-field>

                <v-text-field
                  v-model="userInfo.userEmail"
                  label="이메일"
                  variant="outlined"
                  density="comfortable"
                  required
                  class="mb-3"
                ></v-text-field>

                <v-text-field
                  v-model="userInfo.userPhoneNumber"
                  label="전화번호"
                  variant="outlined"
                  density="comfortable"
                  required
                  class="mb-3"
                ></v-text-field>

                <v-select
                  v-model="userInfo.gender"
                  :items="genderItems"
                  label="성별"
                  variant="outlined"
                  density="comfortable"
                  required
                  class="mb-3"
                ></v-select>
              </div>

              <!-- 제출 버튼 -->
              <div class="text-center">
                <v-btn
                  type="submit"
                  color="orange"
                  size="large"
                  min-width="200"
                  class="mt-4"
                >
                  정보 수정
                </v-btn>
              </div>
            </v-form>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
  </v-container>
</template>

<script>
import axios from "axios";
export default {
  name: "UserUpdate",
  data() {
    return {
      userInfo: {
        userName: "",
        userNickName: "",
        userEmail: "",
        userPhoneNumber: "",
        gender: "",
        profilePhoto: "" // 기존 프로필 사진 URL
      },
      profilePhotoFile: null,
      genderItems: ["MALE", "FEMALE"],
      changePassword: false // 필요 시 비밀번호 변경 여부 추가 가능
    };
  },
  computed: {
    previewImage() {
      if (this.profilePhotoFile) {
        return URL.createObjectURL(this.profilePhotoFile);
      }
      return this.userInfo.profilePhoto || 'default-avatar.png';
    }
  },
  created() {
    this.fetchUserInfo();
  },
  methods: {
    async fetchUserInfo() {
      try {
        const response = await axios.get(`${process.env.VUE_APP_API_BASE_URL}/user/userInfo`, {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("accessToken")}`
          }
        });
        const data = response.data;
        this.userInfo.userName = data.userName;
        this.userInfo.userNickName = data.userNickName;
        this.userInfo.userEmail = data.userEmail;
        this.userInfo.userPhoneNumber = data.userPhoneNumber;
        this.userInfo.gender = data.gender;
        this.userInfo.profilePhoto = data.profilePhoto;
      } catch (error) {
        console.error("유저 정보를 불러오는 데 실패했습니다.", error);
      }
    },
    async updateUser() {
      try {
        const formData = new FormData();
        formData.append("name", this.userInfo.userName);
        formData.append("nickName", this.userInfo.userNickName);
        formData.append("email", this.userInfo.userEmail);
        formData.append("phoneNumber", this.userInfo.userPhoneNumber);
        formData.append("gender", this.userInfo.gender);
        if (this.profilePhotoFile) {
          // ✅ 새로 업로드된 파일이 있으면 그대로 추가
          formData.append("profilePhoto", this.profilePhotoFile);
        } else {
          // ✅ 기존 프로필 사진 유지: URL을 Blob으로 변환 후 File 객체로 FormData에 추가
          if (this.userInfo.profilePhoto) {
            const existingImageFile = await this.urlToFile(this.userInfo.profilePhoto);
            if (existingImageFile) {
              formData.append("profilePhoto", existingImageFile);
            }
          }
        }
        console.log(formData);
        await axios.patch(`${process.env.VUE_APP_API_BASE_URL}/user/update`, formData, {
          headers: {
            "Content-Type": "multipart/form-data",
            Authorization: `Bearer ${localStorage.getItem("accessToken")}`
          }
        });
        // 업데이트 성공 후, 최신 사용자 정보를 받아와 localStorage를 갱신합니다.
        const userResponse = await axios.get(`${process.env.VUE_APP_API_BASE_URL}/user/userInfo`, {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("accessToken")}`
          }
        });
        const updatedData = userResponse.data;
        localStorage.setItem("profilePhoto", updatedData.profilePhoto);
        alert("회원정보가 수정되었습니다.");
        // 라우팅 후 페이지 전체 리로드
        window.location.href = "/user/myPage";
      } catch (error) {
        console.error("회원정보 수정 실패", error);
        alert("회원정보 수정에 실패했습니다.");
      }
    },
    async urlToFile(imageUrl) {
      try {
        const response = await fetch(imageUrl);
        const blob = await response.blob();

        // ✅ Blob을 File 객체로 변환 (File 객체는 FormData에서 `MultipartFile`로 전송 가능)
        return new File([blob], "profilePhoto.jpg", { type: blob.type });
      } catch (error) {
        console.error("🚨 기존 프로필 사진을 File로 변환하는 중 오류 발생:", error);
        return null;
      }
    },
    handleFileChange(event) {
      const file = event.target.files[0];
      if (file) {
        this.profilePhotoFile = file;
      }
    }
  }
};
</script>

<style scoped>
.update-container {
  background-color: #f5f7fa;
  min-height: 100vh;
}

.header-section {
  background: linear-gradient(135deg, #FFA726 0%, #FB8C00 100%);
  color: white;
  padding: 2rem 0;
  position: relative;
}

.form-card {
  background: white;
  border-radius: 16px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.05) !important;
  border: 1px solid #e0e0e0;
}

.form-section {
  border-bottom: 1px solid #e0e0e0;
  padding-bottom: 1.5rem;
}

.form-section:last-child {
  border-bottom: none;
}

.profile-section {
  position: relative;
}

.profile-upload-container {
  position: relative;
  display: inline-block;
}

.hidden-input {
  display: none;
}

.profile-avatar {
  cursor: pointer;
  border: 4px solid white;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  position: relative;
  transition: all 0.3s ease;
}

.profile-avatar:hover .upload-overlay {
  opacity: 1;
}

.upload-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  opacity: 0;
  transition: opacity 0.3s ease;
}

.text-caption {
  font-size: 0.75rem;
  opacity: 0.8;
}

.v-avatar {
  border: 4px solid white;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

:deep(.v-field) {
  border-radius: 8px;
}

:deep(.v-field__outline) {
  border-color: #e0e0e0;
}

:deep(.v-field--focused .v-field__outline) {
  border-color: #FFA726 !important;
}

:deep(.v-field--variant-outlined.v-field--focused .v-field__outline) {
  --v-field-border-width: 2px;
  border-color: #FFA726 !important;
}

:deep(.v-selection-control--density-comfortable.v-selection-control--selected .v-selection-control__wrapper) {
  color: #FFA726 !important;
}

@media (max-width: 600px) {
  .header-section {
    padding: 1.5rem 0;
  }

  .form-card {
    padding: 1rem !important;
  }

  .profile-avatar {
    width: 100px !important;
    height: 100px !important;
  }
}
</style>
