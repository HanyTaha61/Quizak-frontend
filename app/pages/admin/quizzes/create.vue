<template>
  <!-- إضافة dir="rtl" لضمان تنسيق الواجهة من اليمين لليسار -->
  <v-container class="max-width-900 py-10" dir="rtl">
    
    <!-- 🚀 قسم العنوان الرئيسي -->
    <div class="d-flex align-center justify-space-between mb-8">
        <v-btn
          color="primary"
          size="x-large"
          rounded="xl"
          elevation="4"
          prepend-icon="mdi-rocket-launch"
          :disabled="!isQuizValid"
          @click="submitQuiz"
        >
          نشر الاختبار
        </v-btn>
      <div>
        <h1 class="text-h4 font-weight-bold mb-1">مصمم الاختبارات</h1>
        <p class="text-subtitle-1 text-medium-emphasis">قم بتصميم اختبار شخصية يعتمد على المنطق</p>
      </div>
    </div>

    <!-- 🔹 المعلومات الأساسية -->
    <v-card variant="outlined" class="rounded-lg border-opacity-100 mb-8 pa-6 bg-surface">
      <div class="d-flex align-center mb-4">
        <v-icon color="primary" class="ml-2">mdi-information-outline</v-icon>
        <h3 class="text-h6 font-weight-bold">هوية الاختبار</h3>
      </div>

      <v-row>
        <v-col cols="12" md="6">
          <v-text-field 
            v-model="quiz.title" 
            label="عنوان الاختبار" 
            variant="filled" 
            placeholder="مثال: أي شخصية من حرب النجوم أنت؟" 
          />
        </v-col>
        <v-col cols="12" md="6">
          <v-text-field 
            v-model="quiz.description" 
            label="وصف الاختبار" 
            variant="filled" 
            placeholder="اكتب وصفاً مختصراً لاختبارك" 
          />
        </v-col>
        <v-col cols="12" md="6">
          <v-text-field 
            v-model="quiz.slug" 
            label="رابط الاختبار (Slug)" 
            variant="filled" 
            prefix="quizapp.com/" 
          />
        </v-col>
        <v-col cols="12">
          <v-text-field 
            v-model="quiz.image" 
            label="رابط صورة الغلاف" 
            variant="filled" 
            prepend-inner-icon="mdi-image-outline"
          />
        </v-col>
      </v-row>
    </v-card>

    <!-- 🔹 بناء النتائج -->
    <section class="mb-10">
      <div class="d-flex align-center justify-space-between mb-4">
        <div class="d-flex align-center">
          <v-icon color="success" class="ml-2">mdi-trophy-outline</v-icon>
          <h3 class="text-h6 font-weight-bold">النتائج والمخرجات</h3>
        </div>
        <v-btn variant="text" color="primary" prepend-icon="mdi-plus" @click="addResult">إضافة نتيجة</v-btn>
      </div>

      <v-row>
        <v-col v-for="(r, ri) in quiz.results" :key="ri" cols="12" md="6">
          <v-card variant="outlined" class="pa-4 position-relative border-dashed">
            <v-btn 
              icon="mdi-close-circle" 
              variant="text" 
              color="error" 
              size="small" 
              class="position-absolute top-0 left-0 mt-2 ml-2"
              @click="quiz.results.splice(ri, 1)"
            />
            <v-text-field v-model="r.key" label="معرف النتيجة (مثلاً: lion)" variant="underlined" density="compact" />
            <v-text-field v-model="r.title" label="العنوان الظاهر للمستخدم" variant="underlined" density="compact" />
            <v-textarea v-model="r.description" label="وصف النتيجة" variant="underlined" rows="2" />
            <!-- حقل الصورة المذكور في تعليقك -->
            <v-text-field v-model="r.image" label="رابط صورة النتيجة" variant="underlined" density="compact" prepend-inner-icon="mdi-camera" />
          </v-card>
        </v-col>
      </v-row>
      
      <v-empty-state
        v-if="!quiz.results.length"
        text="قم بتعريف النتائج التي يمكن أن يحصل عليها المستخدم أولاً."
        icon="mdi-flask-empty-outline"
        label="لا توجد نتائج بعد"
      />
    </section>

    <!-- 🔹 بناء الأسئلة -->
<!-- 🔹 بناء الأسئلة -->
<section class="mb-10">
  <div class="d-flex align-center justify-space-between mb-4">
    <div class="d-flex align-center">
      <v-icon color="secondary" class="ml-2">mdi-comment-question-outline</v-icon>
      <h3 class="text-h6 font-weight-bold">الأسئلة</h3>
    </div>
    <v-btn color="secondary" rounded="pill" prepend-icon="mdi-plus" @click="addQuestion">سؤال جديد</v-btn>
  </div>

  <v-hover v-for="(q, qi) in quiz.questions" :key="qi" v-slot="{ isHovering, props }">
    <v-card
      v-bind="props"
      class="mb-6 rounded-xl transition-swing"
      :elevation="isHovering ? 8 : 1"
      variant="flat"
      border
    >
      <!-- رأس البطاقة: يحتوي على رقم السؤال وزر الحذف -->
      <div class="bg-secondary-lighten-5 pa-4 d-flex align-center justify-space-between">
        <div class="d-flex align-center flex-grow-1">
          <v-chip size="small" color="secondary" class="ml-3">س{{ qi + 1 }}</v-chip>
          <v-text-field 
            v-model="q.text" 
            placeholder="اكتب سؤالك هنا..." 
            hide-details 
            variant="plain"
            class="text-h6 font-weight-medium"
          />
        </div>
        
        <!-- زر حذف السؤال بالكامل -->
        <v-btn 
          icon="mdi-delete-forever" 
          variant="text" 
          color="error" 
          @click="removeQuestion(qi)"
          title="حذف السؤال"
        />
      </div>

      <v-divider />

      <!-- قسم الخيارات (كما هو) -->
      <div class="pa-6">
        <div v-for="(opt, oi) in q.options" :key="oi" class="d-flex align-center gap-3 mb-3">
          <v-text-field 
            v-model="opt.text" 
            label="نص الخيار" 
            variant="outlined" 
            density="comfortable" 
            hide-details 
          />
          <v-select
            :items="resultKeys"
            v-model="opt.value"
            label="يؤدي إلى النتيجة..."
            variant="outlined"
            density="comfortable"
            hide-details
            style="max-width: 200px"
          />
          <v-btn 
            icon="mdi-delete-outline" 
            variant="text" 
            color="grey" 
            @click="q.options.splice(oi, 1)" 
          />
        </div>
        
        <v-btn 
          variant="tonal" 
          size="small" 
          prepend-icon="mdi-plus" 
          class="mt-2"
          @click="addOption(qi)"
        >
          إضافة خيار
        </v-btn>
      </div>
    </v-card>
  </v-hover>
</section>

  </v-container>
</template>

<script setup>
import { ref, computed } from 'vue'
import { useRouter } from 'vue-router'

const router = useRouter()
const API_BASE_URL = import.meta.env
const quiz = ref({
  title: '',
  description: '',
  slug: '',
  image: '',
  questions: [],
  results: []
})

definePageMeta({
  ssr: false
})

/* 🔥 مفاتيح النتائج الديناميكية */
const resultKeys = computed(() =>
  quiz.value.results.map(r => r.key)
)

/* ---------------- المكونات ---------------- */
const addQuestion = () => {
  quiz.value.questions.push({
    text: '',
    options: [
      { text: '', value: '' },
      { text: '', value: '' }
    ]
  })
}

// delete question
const removeQuestion = (index) => {
  if (confirm('هل أنت متأكد من حذف هذا السؤال نهائياً؟')) {
    quiz.value.questions.splice(index, 1)
  }
}

const addOption = (index) => {
  quiz.value.questions[index].options.push({
    text: '',
    value: ''
  })
}

const addResult = () => {
  quiz.value.results.push({
    key: '',
    title: '',
    description: '',
    image: ''
  })
}

/* ---------------- الإرسال ---------------- */
const submitQuiz = async () => {
  try {
    await $fetch('http://localhost:6500/api/quizzes', {
      method: 'POST',
      body: quiz.value
    })

    alert('✅ تم إنشاء الاختبار بنجاح')
    router.push("/")
  } catch (err) {
    console.error(err)
    alert('❌ حدث خطأ أثناء الحفظ')
  }
}

// التحقق من صحة البيانات قبل النشر
const isQuizValid = computed(() => {
  if (!quiz.value.title || !quiz.value.slug) return false
  if (!quiz.value.results.length) return false
  if (!quiz.value.questions.length) return false

  return quiz.value.questions.every(q =>
    q.text &&
    q.options.length >= 2 &&
    q.options.every(opt => opt.text && opt.value)
  )
})
</script>

<style scoped>
.max-width-900 {
  max-width: 900px;
}
.border-dashed {
  border-style: dashed !important;
  border-width: 2px !important;
}
.gap-3 {
  gap: 12px;
}
.transition-swing {
  transition: all 0.3s cubic-bezier(0.25, 0.8, 0.25, 1);
}
/* تأكد من أن الأيقونات والمسافات تتماشى مع RTL */
:deep(.v-field__prepend-inner) {
  padding-inline-end: 8px;
}
</style>