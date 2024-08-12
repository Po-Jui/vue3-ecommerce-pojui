<template>
  <div>
    <div class="user-banner">
      <h2 v-if="isLogin" class="text-center">會員中心</h2>
      <h2 v-else class="text-center">會員登入</h2>
    </div>
    <div class="container user-member-box" v-if="isLogin">
      <div class="row">
        <div class="col-lg-6 col-md-12 mx-auto mt-3">
          <v-tabs class="mt-3" v-model="tab" grow align-tabs="center" color="light-blue-accent-4">
            <v-tab :value="0" class="font-weight-bold">修改個人資料</v-tab>
            <v-tab :value="1" class="font-weight-bold">歷史訂單查詢</v-tab>
            <v-tab :value="2" class="font-weight-bold" v-if="tabCtrl">修改密碼</v-tab>
          </v-tabs>
          <v-tabs-window v-model="tab"
            ><v-tabs-window-item :value="0">
              <v-container fluid>
                <v-form v-model="valid" ref="profileForm">
                  <!-- 圖片預覽 -->
                  <div @click="triggerFileInput" style="width: 100px">
                    <v-img v-if="member.image" :src="member.image" class="my-4 profile-img"></v-img>
                    <v-img
                      v-else
                      src="https://www.iconpacks.net/icons/2/free-user-camera-icon-3355-thumb.png"
                      class="my-4 profile-img"
                    ></v-img>
                  </div>
                  <!-- 隱藏的文件輸入 -->
                  <v-file-input
                    ref="fileInput"
                    class="d-none"
                    v-model="uploadImageDetail"
                    accept="image/*"
                    @change="onImageChange"
                  ></v-file-input>
                  <v-btn color="red" v-if="member.image" @click="removeImage" class="mb-3">
                    刪除圖片
                  </v-btn>
                  <v-text-field
                    prepend-icon="mdi-account"
                    v-model.trim="member.name"
                    :rules="[rules.required]"
                    maxlength="25"
                    placeholder="請輸入姓名"
                    required
                  >
                    <template v-slot:label>
                      <div class="d-flex align-center justify-space-between">
                        <span>姓名</span>
                        <span class="counter ml-2">{{ member.name.length }}/25</span>
                      </div>
                    </template>
                  </v-text-field>
                  <v-text-field
                    prepend-icon="mdi-email"
                    v-model.trim="member.email"
                    label="電子郵件"
                    disabled
                  >
                    <template v-slot:append>
                      <v-btn color="success" v-if="!tabCtrl">
                        <v-icon>mdi-google</v-icon>
                        已驗證
                      </v-btn>
                      <v-btn color="success" v-if="tabCtrl"> 已驗證 </v-btn>
                    </template></v-text-field
                  >
                  <v-text-field
                    prepend-icon="mdi-cellphone"
                    v-model.trim="member.phone"
                    :rules="[rules.phone]"
                    maxlength="10"
                    placeholder="請輸入手機號碼"
                    required
                  >
                    <template v-slot:label>
                      <div class="d-flex align-center justify-space-between">
                        <span>手機號碼</span>
                        <span class="counter ml-2">{{ member.phone.length }}/10</span>
                      </div>
                    </template>
                  </v-text-field>
                  <v-text-field
                    prepend-icon="mdi-map-marker"
                    v-model.trim="member.address"
                    :rules="[rules.noSpacesOrSpecial]"
                    label="地址"
                    placeholder="請輸入地址"
                    required
                  ></v-text-field>
                  <v-btn class="mb-3 w-100" color="primary" @click="updateMember"> 確認修改 </v-btn>
                  <v-btn block color="danger" @click="logout">登出</v-btn>
                </v-form></v-container
              ></v-tabs-window-item
            >
            <v-tabs-window-item :value="1"
              ><v-container fluid><OrderHistory :ordersInfo="propsItem" /></v-container>
            </v-tabs-window-item>
            <v-tabs-window-item :value="2"
              ><v-container fluid
                ><v-text-field
                  prepend-icon="mdi-lock"
                  label="原密碼"
                  placeholder="請輸入原密碼"
                  type="password"
                  required
                ></v-text-field></v-container></v-tabs-window-item
          ></v-tabs-window>
        </div>
      </div>
    </div>
    <div class="container user-login-box my-5" v-else>
      <div class="row">
        <div class="col-lg-4 col-md-8 mx-auto">
          <loading :active.sync="isLoading">
            <i class="loading-box"></i>
          </loading>
          <div class="mb-5">
            <div class="mb-3">
              <div class="user-login-item">
                <div class="d-flex">
                  <v-card class="elevation-12">
                    <v-tabs grow v-model="tab" align-tabs="center" color="light-blue-accent-4">
                      <v-tab :value="0" class="font-weight-bold">登入</v-tab>
                      <v-tab :value="1" class="font-weight-bold">註冊</v-tab>
                      <v-tab :value="2" class="font-weight-bold">忘記密碼</v-tab>
                    </v-tabs>
                    <v-tabs-window v-model="tab">
                      <v-tabs-window-item :value="0">
                        <v-card class="elevation-12">
                          <v-card-text>
                            <v-form ref="loginForm" v-model="valid" lazy-validation>
                              <v-text-field
                                prepend-icon="mdi-account"
                                v-model="login.account"
                                :rules="[rules.required, rules.email]"
                                label="帳號"
                                type="text"
                                placeholder="請輸入email"
                                required
                              ></v-text-field>
                              <v-text-field
                                :type="showPassword ? 'text' : 'password'"
                                prepend-icon="mdi-lock"
                                v-model="login.password"
                                :rules="[
                                  rules.required,
                                  rules.hasUpperCase,
                                  rules.hasLowerCase,
                                  rules.min,
                                  rules.noSpacesOrSpecial,
                                ]"
                                label="密碼"
                                placeholder="請輸入密碼"
                                required
                              >
                                <template v-slot:append>
                                  <v-icon
                                    @mousedown="togglePasswordVisibility(true)"
                                    @mouseup="togglePasswordVisibility(false)"
                                  >
                                    {{ showPassword ? "mdi-eye" : "mdi-eye-off" }}
                                  </v-icon>
                                </template>
                              </v-text-field>
                            </v-form>
                          </v-card-text>
                          <v-card-actions>
                            <v-spacer></v-spacer>
                            <v-btn color="success" @click.prevent="loginSubmit">登入</v-btn>
                            <v-btn text color="warning" @click="switchTab(2)">忘記密碼?</v-btn>
                          </v-card-actions>
                          <v-divider>第三方登入</v-divider>
                          <v-container class="d-flex justify-center" style="width: 50%">
                            <v-img
                              class="hover-image"
                              :src="googleLogo"
                              alt="Google Logo"
                              @click="loginWithGoogle"
                            />
                          </v-container>
                        </v-card>
                      </v-tabs-window-item>
                      <v-tabs-window-item :value="1">
                        <v-card class="elevation-12">
                          <v-card-text
                            ><v-form ref="registerForm" v-model="valid" lazy-validation>
                              <v-text-field
                                prepend-icon="mdi-account-arrow-left"
                                v-model.trim="register.firstName"
                                :rules="[rules.required, rules.noSpacesOrSpecial]"
                                label="名字"
                                placeholder="請輸入名字"
                                required
                              ></v-text-field>
                              <v-text-field
                                prepend-icon="mdi-account-arrow-right"
                                v-model.trim="register.lastName"
                                :rules="[rules.required, rules.noSpacesOrSpecial]"
                                label="姓氏"
                                placeholder="請輸入姓氏"
                                required
                              ></v-text-field>
                              <v-text-field
                                prepend-icon="mdi-email"
                                v-model.trim="register.email"
                                :rules="[rules.required, rules.email]"
                                label="電子郵件"
                                placeholder="請輸入email"
                                required
                              ></v-text-field>
                              <v-text-field
                                :type="showPassword ? 'text' : 'password'"
                                prepend-icon="mdi-lock"
                                v-model.trim="register.password"
                                :rules="[
                                  rules.required,
                                  rules.hasUpperCase,
                                  rules.hasLowerCase,
                                  rules.min,
                                  rules.noSpacesOrSpecial,
                                ]"
                                label="密碼"
                                placeholder="請輸入密碼"
                                required
                              >
                                <template v-slot:append>
                                  <v-icon
                                    @mousedown="togglePasswordVisibility(true)"
                                    @mouseup="togglePasswordVisibility(false)"
                                  >
                                    {{ showPassword ? "mdi-eye" : "mdi-eye-off" }}
                                  </v-icon>
                                </template>
                              </v-text-field>
                              <v-text-field
                                :type="showconfirmPassword ? 'text' : 'password'"
                                prepend-icon="mdi-lock-check"
                                v-model.trim="register.confirmPassword"
                                :rules="[rules.required, passwordMatch]"
                                label="密碼確認"
                                placeholder="請再輸入一次密碼"
                                required
                              >
                                <template v-slot:append>
                                  <v-icon
                                    @mousedown="toggleConfirmPasswordVisibility(true)"
                                    @mouseup="toggleConfirmPasswordVisibility(false)"
                                  >
                                    {{ showconfirmPassword ? "mdi-eye" : "mdi-eye-off" }}
                                  </v-icon>
                                </template>
                              </v-text-field>
                              <v-card-actions>
                                <v-spacer></v-spacer>
                                <v-btn color="success" @click.prevent="registerSubmit">提交</v-btn>
                              </v-card-actions>
                            </v-form></v-card-text
                          >
                        </v-card>
                      </v-tabs-window-item>
                      <v-tabs-window-item :value="2">
                        <v-card class="elevation-12">
                          <v-card-text>
                            <v-form ref="forgetPassword">
                              <v-text-field
                                prepend-icon="mdi-email"
                                ref="forgetPassword"
                                v-model.trim="forgetPassword"
                                :rules="[rules.required, rules.email]"
                                label="請輸入email"
                              ></v-text-field>
                              <v-btn class="mt-2" type="submit" block @click.prevent="findPassword"
                                >重設密碼</v-btn
                              >
                            </v-form>
                          </v-card-text>
                        </v-card>
                      </v-tabs-window-item>
                    </v-tabs-window>
                  </v-card>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import OrderHistory from "@/components/frontend/OrderHistory.vue";
import googleLogo from "@/assets/google.png";
import Toast from "@/alert/Toast";
import { auth, db, googleprovider } from "@/methods/firebase";
import { collection, doc, updateDoc, getDoc, getDocs, setDoc } from "firebase/firestore";
import {
  createUserWithEmailAndPassword,
  signInWithEmailAndPassword,
  signInWithPopup,
  sendEmailVerification,
  onAuthStateChanged,
  sendPasswordResetEmail,
  updateProfile,
  signOut,
} from "firebase/auth";

export default {
  data() {
    return {
      googleLogo,
      showPassword: false,
      showconfirmPassword: false,
      isLoading: false,
      tab: 0, //  預設為第一個tab
      isLogin: false, // 是否登入
      isRegistering: false, // 判斷是否為註冊
      displayName: "", // 登入後顯示的會員名稱
      userInfo: [], // 資料庫的會員資料
      carts: [],
      // 登入後的會員資料
      member: {
        name: "",
        email: "",
        password: "",
        phone: "",
        address: "",
        image: null,
      },
      uploadImageDetail: null, // 上傳圖片的詳細資訊
      valid: false, // form驗證
      // 登入欄位
      login: {
        account: "",
        password: "",
      },
      // 註冊欄位
      register: {
        firstName: "",
        lastName: "",
        email: "",
        password: "",
        confirmPassword: "",
      },
      forgetPassword: "", // 忘記密碼
      tabCtrl: true,
      propsItem: {},
      rules: {
        noSpacesOrSpecial: (value) => {
          const pattern = /^[^\s!@#$%^&*(),.?":{}|<>]*$/; // Regular expression to disallow spaces and special characters
          return pattern.test(value) || "不得包含空白或特殊符號";
        },
        required: (value) => !!value || "此欄位為必填項",
        email: (value) => {
          const pattern = /^[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,4}$/;
          return pattern.test(value) || "請輸入有效的電子郵件地址";
        },
        phone: (value) => {
          const pattern = /^09\d{8}$/;
          return pattern.test(value) || "請輸入正確的手機號碼";
        },
        min: (value) => value.length >= 6 || "密碼至少需要6個字符",
        max: (value) => value.length <= 25 || "姓名最多25個字符",
        hasUpperCase: (value) => /[A-Z]/.test(value) || "需包含至少一個大寫英文字母",
        hasLowerCase: (value) => /[a-z]/.test(value) || "需包含至少一個小寫英文字母",
      },
      uid: null,
    };
  },
  components: {
    OrderHistory,
  },
  watch: {
    isLogin(newVal, oldVal) {
      if (newVal) {
        this.updateCartItem(this.uid);
        console.log("User is logged in with UID:", newVal);
        // this.updateUserCartItem(); // 可以在這裡調用更新購物車方法
      } else {
        console.log("User is logged out or email is not verified");
        // 在這裡處理用戶登出或未驗證的情況
      }
    },
  },
  created() {
    this.isLoading = true;
    onAuthStateChanged(auth, (user) => {
      if (user) {
        if (user.emailVerified) {
          this.isLogin = true;
          this.uid = user.uid;
          this.displayName = user.displayName;
          this.member.name = user.displayName;
          this.member.email = user.email;
          if (user.providerData[0].providerId === "password") {
            this.tabCtrl = true;
          } else {
            this.tabCtrl = false;
          }
        }
      }
    });
    this.$emitter.on("logout-user", () => {
      this.logout();
      this.isLogin = false;
    });
    this.isLoading = false;
  },
  mounted() {
    onAuthStateChanged(auth, (user) => {
      if (user) {
        if (user.emailVerified) {
          this.getUserInfo(user.uid);
        }
      }
    });
    this.$emitter.on("logout-user", () => {
      this.logout();
    });
  },
  beforeUnmount() {
    this.$emitter.off("logout-user", () => {
      this.logout();
    });
  },
  methods: {
    // 切換tab
    switchTab(index) {
      this.tab = index;
    },
    // 上傳圖片
    triggerFileInput() {
      this.$refs.fileInput.$el.querySelector("input").click();
    },
    onImageChange(event) {
      const file = event.target.files[0];
      if (file) {
        const reader = new FileReader();
        reader.onload = (e) => {
          this.member.image = e.target.result;
        };
        reader.readAsDataURL(file);
      }
    },
    removeImage() {
      this.member.image = null;
      this.uploadImageDetail = null;
    },
    // 修改個人資料
    updateMember() {
      this.$refs.profileForm.validate().then((res) => {
        if (res.valid === false) {
          Toast.fire({
            icon: "error",
            title: "請輸入正確資訊",
          });
        } else {
          updateProfile(auth.currentUser, {
            displayName: this.member.name,
          })
            .then(() => {
              this.displayName = this.member.name;
              this.updateUserInfo(
                auth.currentUser.uid,
                this.member.address,
                this.member.phone,
                this.member.image
              );
              Toast.fire({
                icon: "success",
                title: "修改成功",
              });
            })
            .catch((error) => {
              console.log(`錯誤：${error.message}`);
              Toast.fire({
                icon: "error",
                title: "修改失敗",
              });
            });
        }
      });
    },
    async updateUserInfo(userId, newAddress, newPhoneNumber, newPhotoURL) {
      try {
        // 指定要更新的文檔
        const userDoc = doc(db, "userInfo", userId);
        // 更新文檔數據
        await updateDoc(userDoc, {
          address: newAddress,
          phoneNumber: newPhoneNumber,
          photoURL: newPhotoURL,
        });
        this.$emitter.emit("update-user-login-photo", this.member);
      } catch (error) {
        console.error("Error updating document: ", error);
      }
    },
    // 從cloud firestore 獲取 userInfo 集合中的所有文件
    async getUserInfo(uid) {
      try {
        // 指定集合
        const userInfoCollection = collection(db, "userInfo");
        // 獲得集合中的所有文件
        const querySnapshot = await getDocs(userInfoCollection);
        this.userInfo = [];
        querySnapshot.forEach((doc) => {
          let appData = doc.data();
          appData.id = doc.id;
          this.userInfo.push({
            id: appData.id,
            address: appData.address,
            phoneNumber: appData.phoneNumber,
            photoURL: appData.photoURL,
            orders: appData.orders,
          });
        });
        this.userInfo.forEach((item) => {
          if (item.id === uid) {
            this.propsItem = item.orders;
            this.member.address = item.address;
            this.member.phone = item.phoneNumber;
            this.member.image = item.photoURL;
          }
        });
      } catch (error) {
        console.error("Error fetching user info:", error);
      }
    },
    // 顯示密碼
    togglePasswordVisibility() {
      this.showPassword = !this.showPassword;
    },
    toggleConfirmPasswordVisibility() {
      this.showconfirmPassword = !this.showconfirmPassword;
    },
    // 登入
    loginSubmit() {
      this.isLoading = true;
      this.isRegistering = false;
      this.$refs.loginForm.validate().then((res) => {
        if (res.valid === false) {
          Toast.fire({
            icon: "error",
            title: "請輸入正確資訊",
          });
        } else {
          // Firebase API 傳入auth物件、信箱和密碼 可登入
          signInWithEmailAndPassword(auth, this.login.account, this.login.password)
            .then((res) => {
              this.getAuthState();
            })
            .catch((error) => {
              console.log(`錯誤：${error.message}`);
              switch (error.message) {
                case "Firebase: Error (auth/user-not-found).":
                  Toast.fire({
                    icon: "error",
                    title: "帳號不存在",
                  });
                  break;
                case "Firebase: Error (auth/wrong-password).":
                  Toast.fire({
                    icon: "error",
                    title: "密碼錯誤",
                  });
                  break;
                case "Firebase: Error (auth/invalid-credential).":
                  Toast.fire({
                    icon: "error",
                    title: "登入失敗，帳號或密碼錯誤",
                  });
                  break;
                default:
                  Toast.fire({
                    icon: "error",
                    title: "登入失敗",
                  });
              }
            });
        }
      });
      this.isLoading = false;
    },
    getAuthState() {
      onAuthStateChanged(auth, (user) => {
        // 如果使用者有登入(包括剛註冊帳號) user參數會是true
        if (user) {
          //判斷是否已經信箱註冊
          if (user.emailVerified) {
            this.login = { account: "", password: "" };
            Toast.fire({
              icon: "success",
              title: `${user.displayName}歡迎回來`,
            });
            this.isLogin = true;
            this.displayName = user.displayName;
            this.member.name = user.displayName;
            this.member.email = user.email;
            this.$emitter.emit("update-user-status"); // 更新登入狀態
          } else if (this.isRegistering === false && user.emailVerified === false) {
            Toast.fire({
              icon: "info",
              title: "請至信箱驗證帳號",
            });
          }
        }
      });
    },
    // 第三方登入
    async loginWithGoogle() {
      try {
        const result = await signInWithPopup(auth, googleprovider);
        const user = result.user;
        this.checkDocExistsMixin(user);
        Toast.fire({
          icon: "success",
          title: `${user.displayName}歡迎回來`,
        });
      } catch (error) {
        console.error("Error during Google login:", error);
      }
    },
    // 確認文件是否存在
    async checkDocExists(docId) {
      try {
        // 指定文檔路徑
        const docRef = doc(db, "userInfo", docId);
        // 獲得文檔有哪些
        const docSnap = await getDoc(docRef);
        // 檢查文檔是否存在
        if (docSnap.exists()) {
          return true;
        } else {
          return false;
        }
      } catch (error) {
        console.error("Error checking document existence: ", error);
        return false;
      }
    },
    checkDocExistsMixin(user) {
      const checkDocExists = this.checkDocExists(user.uid);
      checkDocExists
        .then((result) => {
          if (result === false) {
            this.member.image = user.photoURL;
            this.addUserInfoWithCustomId(
              user.uid,
              this.member.address,
              this.member.phone,
              this.member.image
            );
            this.$emitter.emit("update-user-login-photo", this.member);
          }
        })
        .catch((error) => {
          console.error("Promise rejected with:", error);
        });
    },
    async updateCartItem(uid) {
      const carts = await this.getCart();
      if (carts.length === 0) {
        const url = `${process.env.VUE_APP_API}api/${process.env.VUE_APP_PATH}/cart`;
        const userRef = collection(db, "userInfo");
        const userDocRef = doc(userRef, uid);
        const docSnap = await getDoc(userDocRef);
        if (docSnap.exists()) {
          const userCartItems = docSnap.data().cartItem;
          for (const item of userCartItems) {
            const cart = {
              product_id: item.product.id,
              qty: item.qty,
            };
            try {
              // 發送 POST 請求，將 cart 資料傳送至伺服器
              await this.$http.post(url, { data: cart });
              // 刪除已處理的項目
              // 使用 filter 來創建一個新的陣列，排除已處理的項目
              userCartItems = userCartItems.filter((i) => i !== item);
            } catch (error) {
              // console.error("發送請求時發生錯誤:", error);
            }
          }
          // 當所有項目處理完成後，發送事件以更新總計
          this.$emitter.emit("update-total");
          console.log("userCartItems:", userCartItems);
        } else {
          // doc.data() will be undefined in this case
          console.log("No such document!");
        }
      }
    },
    async getCart() {
      const url = `${process.env.VUE_APP_API}api/${process.env.VUE_APP_PATH}/cart`;
      this.isLoading = true;
      let carts = [];
      try {
        const response = await this.$http.get(url);
        carts = response.data.data.carts;
      } catch (error) {
        Toast.fire({
          title: "資料讀取失敗，請稍後再試",
          icon: "error",
        });
      } finally {
        this.isLoading = false;
      }
      return carts;
    },
    // 註冊
    passwordMatch(value) {
      return value === this.register.password || "密碼不匹配";
    },
    registerSubmit() {
      this.isLoading = true;
      this.isRegistering = true;
      this.$refs.registerForm.validate().then((res) => {
        if (res.valid === false) {
          Toast.fire({
            icon: "error",
            title: "請輸入正確資訊",
          });
        } else {
          // Firebase API 傳入auth物件、信箱和密碼 可創立帳號
          createUserWithEmailAndPassword(auth, this.register.email, this.register.password)
            .then((res) => {
              const user = res.user;
              updateProfile(user, {
                displayName: this.register.firstName + " " + this.register.lastName,
              }).then(() => {
                // 設定認證信箱的基本設定
                auth.languageCode = "zh-TW";
                // Firebase API 會寄認證信給指定信箱 要把auth.currentUser傳入
                sendEmailVerification(auth.currentUser).then(() => {
                  Toast.fire({
                    icon: "success",
                    title: "註冊成功，驗證信已發送到您的信箱，請點選信中驗證連結",
                  });
                });
              });
              this.addUserInfoWithCustomId(user.uid);
              this.register = {
                email: "",
                password: "",
                firstName: "",
                lastName: "",
                confirmPassword: "",
              };
              this.$refs.registerForm.reset();
              this.switchTab(0);
            })
            .catch((error) => {
              console.log(`錯誤：${error.message}`);
              switch (error.message) {
                case "Firebase: Error (auth/email-already-in-use).":
                  Toast.fire({
                    icon: "error",
                    title: "帳號已註冊，請到登入頁面登入",
                  });
                  break;
                default:
                  Toast.fire({
                    icon: "error",
                    title: "註冊失敗",
                  });
                  break;
              }
            });
        }
      });
      this.isLoading = false;
    },
    // 增加新用户的 uid 到 userInfo 集合
    async addUserInfoWithCustomId(userId) {
      try {
        // 指定集合和文檔 ID
        const userDoc = doc(db, "userInfo", userId);

        // 添加文檔數據
        await setDoc(userDoc, {
          address: "",
          phoneNumber: "",
          photoURL: null,
          orders: [],
          cartItem: [],
          favorite: [],
        });
      } catch (error) {
        console.error("Error adding document: ", error);
      }
    },
    findPassword() {
      this.$refs.forgetPassword.validate().then((res) => {
        if (res.errors.length > 0) {
          Toast.fire({
            icon: "error",
            title: "請輸入正確資訊",
          });
        } else {
          sendPasswordResetEmail(auth, this.forgetPassword)
            .then(() => {
              Toast.fire({
                icon: "success",
                title:
                  "如果您的信箱已註冊，我們將發送密碼重置郵件給您，請至您的信箱點選信中連結設定新密碼",
              });
              this.forgetPassword = "";
              this.switchTab(0);
              this.$refs.forgetPassword.reset();
            })
            .catch((error) => {
              console.log(`錯誤：${error.message}`);
              switch (error.message) {
                case "Firebase: Error (auth/missing-email).":
                  Toast.fire({
                    icon: "error",
                    title: "請輸入正確信箱",
                  });
                  break;
              }
            });
        }
      });
    },
    logout() {
      this.$swal({
        title: "確定要登出嗎?",
        icon: "warning",
        showCancelButton: true,
        scrollbarPadding: false,
        confirmButtonColor: "#3085d6",
        cancelButtonColor: "#d33",
        confirmButtonText: "是",
        cancelButtonText: "否",
      }).then((result) => {
        if (result.isConfirmed) {
          this.isLoading = true;
          signOut(auth)
            .then(() => {
              this.isLoading = false;
              this.isLogin = false;
              this.tab = 0;
              this.removeAllCartItem();
              Toast.fire({
                icon: "success",
                title: `${this.displayName}貴賓，期待您的再次光臨`,
              });
              this.$emitter.emit("update-user-status"); // 更新登入狀態
              this.member = {
                name: "",
                email: "",
                password: "",
                phone: "",
                address: "",
                image: null,
              };
            })
            .catch(() => {
              Toast.fire({
                icon: "error",
                title: "發生錯誤，請重新整理頁面",
              });
            });
        }
      });
    },
    removeAllCartItem() {
      const url = `${process.env.VUE_APP_API}api/${process.env.VUE_APP_PATH}/carts`;
      this.$http
        .delete(url)
        .then(() => {
          this.$emitter.emit("update-total");
        })
        .catch((err) => {
          console.error(err);
        });
    },
  },
};
</script>

<style lang="scss" scoped>
.elevation-12 {
  margin-top: 10px;
  width: 425px;
}
:deep(.v-input__details) {
  display: initial !important;
}
:deep(.v-input--horizontal .v-input__append) {
  margin-inline-start: -30px;
}
@media (max-width: 455px) {
  .elevation-12 {
    width: 405px;
  }
}
@media (max-width: 435px) {
  .elevation-12 {
    width: 385px;
  }
}
@media (max-width: 415px) {
  .elevation-12 {
    width: 365px;
  }
}
@media (max-width: 385px) {
  .elevation-12 {
    width: 345px;
  }
}
@media (max-width: 365px) {
  .elevation-12 {
    width: 325px;
  }
}
</style>
