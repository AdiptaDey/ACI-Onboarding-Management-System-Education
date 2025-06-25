<script setup>
import MainLayout from "@/components/MainLayout.vue";
import { showNotification } from "@/utilities/notification";
import { onMounted, ref, computed } from "vue";
import { useRoute, useRouter } from "vue-router";
import { apiBase } from "@/config";
import Cookies from "js-cookie";
import axios from "axios";
import {
  sscList,
  gradeList,
  hscList,
  subjectList,
  bscGradeList,
  graduationList,
  mastersList,
  boardList,
  institutionList,
  groupList,
} from "@/stores/data";
const route = useRoute();
const router = useRouter();
const id = localStorage.getItem("staff_id");
const other_board_ssc = ref("");
const other_group_ssc = ref("");
const other_board_hsc = ref("");
const other_group_hsc = ref("");
const other_subject_graduation = ref("");
const other_university_graduation = ref("");
const other_degree_masters = ref("");
const other_subject_masters = ref("");
const other_university_masters = ref("");

const formData = ref({
  education_info: {
    ssc: {
      degree: null,
      institute_name: "",
      board: null,
      group: null,
      result: null,
      cgpa: null,
      passing_year: null,
      achievements: "",
    },
    hsc: {
      degree: null,
      institute_name: "",
      board: null,
      group: null,
      result: null,
      cgpa: null,
      passing_year: null,
      achievements: "",
    },
    graduation: {
      degree: null,
      institute_name: null,
      group: null,
      result: null,
      cgpa: null,
      passing_year: null,
      achievements: "",
    },
    masters: {
      degree: null,
      institute_name: null,
      group: null,
      result: null,
      cgpa: null,
      passing_year: null,
      achievements: null,
    },
    otherone: {
      degree: null,
      institute_name: null,
      group: null,
      result: null,
      cgpa: null,
      board: null,
      boardOther: "",
      passing_year: null,
      achievements: null,
    },
    othertwo: {
      degree: null,
      institute_name: null,
      group: null,
      result: null,
      cgpa: null,
      board: null,
      boardOther: "",
      passing_year: null,
      achievements: null,
    },
  },
  staff_id: localStorage.getItem("staff_id"),
});

const isLoading = ref(true);
const isError = ref(false);
const isExperienced = ref("");

const mapEducationToForm = (education) => {
  switch (education.degree) {
    case "S.S.C":
    case "Dakhil":
    case "S.S.C Vocational":
    case "O Level/Cambridge":
    case "SSC Equivalent":
    case "Dakhil Vocational":
      formData.value.education_info.ssc = { ...education };
      break;
    case "H.S.C":
    case "Alim":
    case "Business Management":
    case "Diploma in Engineering":
    case "A Level/Sr. Cambridge":
    case "H.S.C Equivalent":
    case "Diploma in Pharmacy":
      formData.value.education_info.hsc = { ...education };
      break;

    case "B.B.A":
    case "B. Com":
    case "B.Sc Engineering":
    case "B.Sc in Agricultural Science":
    case "BA":
    case "BBS":
    case "BSS":
    case "Fazil":
    case "Graduation Equivalent":
    case "Honors":
    case "L.L.B":
    case "M.B.B.S/B.D.S":
    case "Pass Course":
    case "bachelor":
      formData.value.education_info.graduation = { ...education };
      break;

    case "M.B.A":
    case "M. Com":
    case "M.Sc Engineering":
    case "M.Sc in Agricultural Science":
    case "MA":
    case "MBS":
    case "MSS":
    case "Kamil":
    case "Masters Equivalent":
    case "Honors":
    case "L.L.M":
    case "Other":
      formData.value.education_info.masters = { ...education };
      break;

    case "JSC":
    case "JDC":
      formData.value.education_info.otherone = { ...education };
      break;
    case "PSC":
      formData.value.education_info.othertwo = { ...education };
      break;
    default:
      console.warn(`Unknown degree: ${education.degree}`);
  }
};

const fetchEducation = async (id) => {
  isLoading.value = true;
  isError.value = false;
  try {
    const token = localStorage.getItem("access_token");
    const config = {
      headers: {
        Authorization: `Bearer ${token}`,
      },
    };
    const { data } = await axios.get(`${apiBase}/educations/${id}`, config);
    if (data && Array.isArray(data)) {
      data.forEach((education) => {
        mapEducationToForm(education);
      });

      // console.log("->>", data);
    } else {
      console.error("Unexpected data structure", data);
    }
  } catch (err) {
    isError.value = true;
    console.error("Failed to fetch references data:", err);
  } finally {
    isLoading.value = false;
  }
};

onMounted(async () => {
  await fetchEducation(id);
});

const isSSCFilled = computed(() => {
  const ssc = formData.value.education_info.ssc;
  return (
    ssc.institute_name ||
    ssc.degree ||
    ssc.group ||
    ssc.board ||
    ssc.result ||
    ssc.passing_year
  );
});

const isHSCFilled = computed(() => {
  const hsc = formData.value.education_info.hsc;
  return (
    hsc.institute_name ||
    hsc.degree ||
    hsc.board ||
    hsc.group ||
    hsc.result ||
    hsc.passing_year
  );
});

const isGraduationFilled = computed(() => {
  const graduation = formData.value.education_info.graduation;
  return (
    graduation.institute_name ||
    graduation.degree ||
    graduation.group ||
    graduation.result ||
    graduation.passing_year
  );
});
const isMastersFilled = computed(() => {
  const masters = formData.value.education_info.masters;
  return (
    masters.institute_name ||
    masters.degree ||
    masters.group ||
    masters.result ||
    masters.passing_year
  );
});

const isOtherone = computed(() => {
  const otherone = formData.value.education_info.otherone;
  return (
    otherone.institute_name ||
    otherone.degree ||
    otherone.group ||
    otherone.board ||
    otherone.result ||
    otherone.passing_year
  );
});

const isOthertwo = computed(() => {
  const othertwo = formData.value.education_info.othertwo;
  return (
    othertwo.institute_name ||
    othertwo.degree ||
    othertwo.group ||
    othertwo.result ||
    othertwo.passing_year
  );
});

const requiredFields = computed(() => {
  return {
    ssc: isSSCFilled.value,
    hsc: isHSCFilled.value,
    graduation: isGraduationFilled.value,
    masters: isMastersFilled.value,
    otherone: isOtherone.value,
    othertwo: isOthertwo.value,
  };
});

const handleSubmit = async (router) => {
  isLoading.value = true;
  try {
    // check ssc
    if (requiredFields.value.ssc) {
      const ssc = formData.value.education_info.ssc;
      if (
        !ssc.institute_name ||
        !ssc.degree ||
        !ssc.board ||
        !ssc.group ||
        !ssc.result ||
        !ssc.passing_year
      ) {
        showNotification(
          "error",
          "Please complete all SSC fields. এসএসসির সব তথ্য পূরণ করুন।"
        );
        return;
      }
    }
    // check hsc
    if (requiredFields.value.hsc) {
      const hsc = formData.value.education_info.hsc;
      if (
        !hsc.institute_name ||
        !hsc.degree ||
        !hsc.board ||
        !hsc.group ||
        !hsc.result ||
        !hsc.passing_year
      ) {
        showNotification(
          "error",
          "Please complete all HSC fields. এইচএসসির সব তথ্য পূরণ করুন।"
        );
        return;
      }
    }

    // check graduation
    if (requiredFields.value.graduation) {
      const graduation = formData.value.education_info.graduation;
      if (
        !graduation.institute_name ||
        !graduation.degree ||
        !graduation.result ||
        !graduation.group ||
        !graduation.passing_year
      ) {
        showNotification(
          "error",
          "Please complete all Graduation fields.  স্নাতকের সব তথ্য পূরণ করুন।"
        );
        return;
      }
    }
    // check masters
    if (requiredFields.value.masters) {
      const masters = formData.value.education_info.masters;
      if (
        !masters.institute_name ||
        !masters.degree ||
        !masters.result ||
        !masters.group ||
        !masters.passing_year
      ) {
        showNotification(
          "error",
          "Please complete all Masters fields. মাস্টার্সের সব তথ্য পূরণ করুন।"
        );
        return;
      }
    }

    // if (requiredFields.value.otherone) {
    //   const otherone = formData.value.education_info.otherone;
    //   if (
    //     !otherone.institute_name ||
    //     !otherone.degree ||
    //     !otherone.group ||
    //     !otherone.board ||
    //     !otherone.result ||
    //     !otherone.passing_year
    //   ) {
    //     showNotification(
    //       "error",
    //       "Please complete all Below SSC / Other Than Above ( One )"
    //     );
    //     return;
    //   }
    // }

    // if (requiredFields.value.othertwo) {
    //   const othertwo = formData.value.education_info.othertwo;
    //   if (
    //     !othertwo.institute_name ||
    //     !othertwo.degree ||
    //     !othertwo.group ||
    //     !othertwo.board ||
    //     !othertwo.result ||
    //     !othertwo.passing_year
    //   ) {
    //     showNotification(
    //       "error",
    //       "Please complete all Below SSC / Other Than Above ( Two )"
    //     );
    //     return;
    //   }
    // }

    if (formData.value.education_info.ssc.board == "Other") {
      formData.value.education_info.ssc.board = other_board_ssc.value;
    }
    if (formData.value.education_info.ssc.group == "Other") {
      formData.value.education_info.ssc.group = other_group_ssc.value;
    }

    if (formData.value.education_info.hsc.board == "Other") {
      formData.value.education_info.hsc.board = other_board_hsc.value;
    }

    if (formData.value.education_info.hsc.group == "Other") {
      formData.value.education_info.hsc.group = other_group_hsc.value;
    }

    if (formData.value.education_info.graduation.institute_name == "Other") {
      formData.value.education_info.graduation.institute_name =
        other_university_graduation.value;
    }

    if (formData.value.education_info.graduation.group == "Other") {
      formData.value.education_info.graduation.group =
        other_subject_graduation.value;
    }

    if (formData.value.education_info.masters.degree == "Other") {
      formData.value.education_info.masters.degree = other_degree_masters.value;
    }

    if (formData.value.education_info.masters.group == "Other") {
      formData.value.education_info.masters.group = other_subject_masters.value;
    }

    if (formData.value.education_info.masters.institute_name == "Other") {
      formData.value.education_info.masters.institute_name =
        other_university_masters.value;
    }

    const token = localStorage.getItem("access_token");
    const config = {
      headers: {
        Authorization: `Bearer ${token}`,
      },
    };
    const response = await axios.post(
      `${apiBase}/educations`,
      formData.value,
      config
    );

    if (response.data.success) {
      showNotification("success", response.data.message);
      router.push({ name: "other-information" });
    } else {
      showNotification("error", response.data.message || "Submission failed.");
    }
  } catch (error) {
    console.error("Error submitting form:", error);
    showNotification("error", "Error occurred while submitting.");
  } finally {
    isLoading.value = false;
  }
};

const startYear = 1950;
const currentYear = new Date().getFullYear();
const yearList = ref([]);

for (let year = startYear; year <= currentYear; year++) {
  yearList.value.push({ value: year.toString() });
}
</script>

<template>
  <MainLayout>
    <fieldset class="border border-gray-300 rounded-lg p-4">
      <legend class="font-medium mb-1 uppercase text-cyan-600 px-3">
        <img
          src="@/assets/images/Education.gif"
          class="inline-block w-14"
        />Education
      </legend>
      <form @keydown.enter.prevent @submit.prevent="handleSubmit($router)">
        <!-- SSC -->
        <div class="mb-8 education overflow-hidden">
          <div class="ssc mb-3 w-full">
            <label class="block mb-1"
              >Select SSC/Equivalent (এসএসসি/সমমান)</label
            >

            <div class="grid grid-cols-3">
              <label
                >School Name :

                <span class="text-red-500">*</span>
              </label>

              <a-input
                :required="requiredFields.ssc"
                class="w-full mb-2 col-span-2"
                allowClear
                placeholder="School Name"
                v-model:value="formData.education_info.ssc.institute_name"
              />
            </div>

            <div class="grid grid-cols-3">
              <label
                >Degree :

                <span class="text-red-500">*</span>
              </label>
              <a-select
                :required="requiredFields.ssc"
                class="w-full mb-2 col-span-2"
                placeholder="Select SSC/Equivalent"
                allowClear
                :options="sscList"
                v-model:value="formData.education_info.ssc.degree"
              ></a-select>
            </div>

            <div class="grid grid-cols-3">
              <label
                >Board :

                <span class="text-red-500">*</span>
              </label>

              <a-select
                :required="requiredFields.ssc"
                class="w-full mb-2 col-span-2"
                placeholder="Select Board"
                allowClear
                :options="boardList"
                v-model:value="formData.education_info.ssc.board"
              ></a-select>
            </div>

            <div
              class="grid grid-cols-3"
              v-if="formData.education_info.ssc.board === 'Other'"
            >
              <label
                >Other Board :

                <span class="text-red-500">*</span>
              </label>
              <a-input
                :required="requiredFields.ssc"
                class="w-full mb-2 col-span-2"
                allowClear
                placeholder="Enter Board..."
                v-model:value="other_board_ssc"
              />
            </div>

            <div class="grid grid-cols-3">
              <label
                >Major/Group :

                <span class="text-red-500">*</span>
              </label>
              <a-select
                :required="requiredFields.ssc"
                class="w-full mb-2 col-span-2"
                placeholder="Select Major/Group"
                allowClear
                :options="groupList"
                v-model:value="formData.education_info.ssc.group"
              ></a-select>
            </div>

            <div
              class="grid grid-cols-3"
              v-if="formData.education_info.ssc.group === 'Other'"
            >
              <label>Other Group :</label>
              <a-input
                :required="requiredFields.ssc"
                class="w-full mb-2 col-span-2"
                allowClear
                placeholder="Enter Group..."
                v-model:value="other_group_ssc"
              />
            </div>

            <div class="grid grid-cols-3">
              <label
                >Result :
                <span class="text-red-500">*</span>
              </label>
              <a-select
                :required="requiredFields.ssc"
                class="w-full mb-2 col-span-2"
                placeholder="Select Result"
                allowClear
                :options="gradeList"
                v-model:value="formData.education_info.ssc.result"
              ></a-select>
            </div>

            <div
              class="grid grid-cols-3"
              v-if="
                ['GPA(out of 4)', 'GPA(out of 5)'].includes(
                  formData.education_info.ssc.result
                )
              "
            >
              <label
                >CGPA/GPA :
                <span class="text-red-500">*</span>
              </label>
              <a-input
                :required="requiredFields.ssc"
                required
                type="text"
                class="w-full mb-2 col-span-2"
                allowClear
                placeholder="CGPA/GPA"
                v-model:value="formData.education_info.ssc.cgpa"
              />
            </div>

            <div class="grid grid-cols-3">
              <label
                >Passing Year :
                <span class="text-red-500">*</span>
              </label>
              <a-select
                :required="requiredFields.ssc"
                class="w-full mb-2 col-span-2"
                placeholder="Select Passing Year"
                allowClear
                :options="yearList"
                v-model:value="formData.education_info.ssc.passing_year"
              ></a-select>
            </div>

            <div class="grid grid-cols-3">
              <label>Achievements :</label>
              <a-input
                class="w-full mb-2 col-span-2"
                allowClear
                placeholder="Achievements (Academic, Scholarship, Distinctions, etc.)"
                v-model:value="formData.education_info.ssc.achievements"
              />
            </div>
          </div>
        </div>

        <!-- HSC -->
        <div class="mb-8 education overflow-hidden">
          <label class="block mb-1"
            >Select HSC/Equivalent (এইচএসসি/সমমান)</label
          >

          <div class="grid grid-cols-3">
            <label
              >College Name :

              <span class="text-red-500">*</span>
            </label>
            <a-input
              :required="requiredFields.hsc"
              class="w-full mb-2 col-span-2"
              allowClear
              placeholder="College Name"
              v-model:value="formData.education_info.hsc.institute_name"
            />
          </div>

          <div class="grid grid-cols-3">
            <label
              >Degree :
              <span class="text-red-500">*</span>
            </label>
            <a-select
              :required="requiredFields.hsc"
              class="w-full mb-2 col-span-2"
              placeholder="Select HSC/Equivalent"
              allowClear
              :options="hscList"
              v-model:value="formData.education_info.hsc.degree"
            ></a-select>
          </div>

          <div class="grid grid-cols-3">
            <label
              >Board :
              <span class="text-red-500">*</span>
            </label>
            <a-select
              :required="requiredFields.hsc"
              class="w-full mb-2 col-span-2"
              placeholder="Select Board"
              allowClear
              :options="boardList"
              v-model:value="formData.education_info.hsc.board"
            ></a-select>
          </div>

          <div
            class="grid grid-cols-3"
            v-if="formData.education_info.hsc.board === 'Other'"
          >
            <label
              >Other Board :
              <span class="text-red-500">*</span>
            </label>
            <a-input
              :required="requiredFields.hsc"
              class="w-full mb-2 col-span-2"
              allowClear
              placeholder="Enter Board..."
              v-model:value="other_board_hsc"
            />
          </div>

          <div class="grid grid-cols-3">
            <label
              >Major/Group :
              <span class="text-red-500">*</span>
            </label>
            <a-select
              :required="requiredFields.hsc"
              class="w-full mb-2 col-span-2"
              placeholder="Select Major/Group"
              allowClear
              :options="groupList"
              v-model:value="formData.education_info.hsc.group"
            ></a-select>
          </div>

          <div
            class="grid grid-cols-3"
            v-if="formData.education_info.hsc.group === 'Other'"
          >
            <label
              >Other Group :
              <span class="text-red-500">*</span>
            </label>
            <a-input
              :required="requiredFields.hsc"
              class="w-full mb-2 col-span-2"
              allowClear
              placeholder="Enter Group..."
              v-model:value="other_group_hsc"
            />
          </div>

          <div class="grid grid-cols-3">
            <label
              >Result :
              <span class="text-red-500">*</span>
            </label>
            <a-select
              :required="requiredFields.hsc"
              class="w-full mb-2 col-span-2"
              placeholder="Select Result"
              allowClear
              :options="gradeList"
              v-model:value="formData.education_info.hsc.result"
            ></a-select>
          </div>

          <div
            class="grid grid-cols-3"
            v-if="
              ['GPA(out of 4)', 'GPA(out of 5)'].includes(
                formData.education_info.hsc.result
              )
            "
          >
            <label
              >CGPA/GPA :
              <span class="text-red-500">*</span>
            </label>
            <a-input
              :required="requiredFields.hsc"
              type="text"
              class="w-full mb-2 col-span-2"
              allowClear
              placeholder="CGPA/GPA"
              v-model:value="formData.education_info.hsc.cgpa"
            />
          </div>

          <div class="grid grid-cols-3">
            <label
              >Passing Year :
              <span class="text-red-500">*</span>
            </label>
            <a-select
              :required="requiredFields.hsc"
              class="w-full mb-2 col-span-2"
              placeholder="Select Passing Year"
              allowClear
              :options="yearList"
              v-model:value="formData.education_info.hsc.passing_year"
            ></a-select>
          </div>

          <div class="grid grid-cols-3">
            <label>Achievements :</label>
            <a-input
              class="w-full mb-2 col-span-2"
              allowClear
              placeholder="Achievements (Academic, Scholarship, Distinctions, etc.)"
              v-model:value="formData.education_info.hsc.achievements"
            />
          </div>
        </div>

        <!-- Graduation -->
        <div class="mb-8 education overflow-hidden">
          <label class="block mb-1"
            >Select Graduation/Equivalent (স্নাতক/সমমান)</label
          >

          <div class="grid grid-cols-3">
            <label
              >Degree :
              <span class="text-red-500">*</span>
            </label>
            <a-select
              :required="requiredFields.graduation"
              placeholder="Select Graduation/Equivalent"
              class="w-full mb-2 col-span-2"
              allowClear
              :options="graduationList"
              v-model:value="formData.education_info.graduation.degree"
            ></a-select>
          </div>

          <div class="grid grid-cols-3">
            <label
              >Subject/Department :
              <span class="text-red-500">*</span>
            </label>
            <a-select
              :required="requiredFields.graduation"
              placeholder="Select Subject/Department"
              class="w-full mb-2 col-span-2"
              allowClear
              show-search
              :options="subjectList"
              v-model:value="formData.education_info.graduation.group"
            ></a-select>
          </div>

          <div
            class="grid grid-cols-3"
            v-if="formData.education_info.graduation.group === 'Other'"
          >
            <label
              >Other Subject :
              <span class="text-red-500">*</span>
            </label>
            <a-input
              :required="requiredFields.graduation"
              class="w-full mb-2 col-span-2"
              allowClear
              placeholder="Enter Subject..."
              v-model:value="other_subject_graduation"
            />
          </div>

          <div class="grid grid-cols-3">
            <label
              >University :
              <span class="text-red-500">*</span>
            </label>
            <a-select
              :required="requiredFields.graduation"
              placeholder="Select University"
              class="w-full mb-2 col-span-2"
              allowClear
              show-search
              :options="institutionList"
              v-model:value="formData.education_info.graduation.institute_name"
            ></a-select>
          </div>

          <div
            class="grid grid-cols-3"
            v-if="formData.education_info.graduation.institute_name === 'Other'"
          >
            <label
              >Other University :
              <span class="text-red-500">*</span>
            </label>
            <a-input
              :required="requiredFields.graduation"
              class="w-full mb-2 col-span-2"
              allowClear
              placeholder="Enter Other University ..."
              v-model:value="other_university_graduation"
            />
          </div>

          <div class="grid grid-cols-3">
            <label
              >Result :
              <span class="text-red-500">*</span>
            </label>
            <a-select
              :required="requiredFields.graduation"
              placeholder="Select Result"
              class="w-full mb-2 col-span-2"
              allowClear
              :options="bscGradeList"
              v-model:value="formData.education_info.graduation.result"
            ></a-select>
          </div>

          <div
            class="grid grid-cols-3"
            v-if="
              ['CGPA(out of 4)', 'CGPA(out of 5)'].includes(
                formData.education_info.graduation.result
              )
            "
          >
            <label
              >CGPA/GPA :
              <span class="text-red-500">*</span>
            </label>
            <a-input
              :required="requiredFields.graduation"
              type="text"
              class="w-full mb-2 col-span-2"
              allowClear
              placeholder="CGPA/GPA"
              v-model:value="formData.education_info.graduation.cgpa"
            />
          </div>

          <div class="grid grid-cols-3">
            <label
              >Passing Year :
              <span class="text-red-500">*</span>
            </label>
            <a-select
              :required="requiredFields.graduation"
              placeholder="Select Passing Year"
              class="w-full mb-2 col-span-2"
              allowClear
              :options="yearList"
              v-model:value="formData.education_info.graduation.passing_year"
            ></a-select>
          </div>

          <div class="grid grid-cols-3">
            <label>Achievements :</label>
            <a-input
              class="w-full mb-2 col-span-2"
              allowClear
              placeholder="Achievements (Academic, Scholarship, Distinctions, etc.)"
              v-model:value="formData.education_info.graduation.achievements"
            />
          </div>
        </div>

        <!-- Masters -->
        <div class="mb-8 education overflow-hidden">
          <label class="block mb-1"
            >Select Masters/Equivalent (মাস্টার্স/সমমান)</label
          >

          <div class="grid grid-cols-3">
            <label
              >Degree :
              <span class="text-red-500">*</span>
            </label>
            <a-select
              :required="requiredFields.masters"
              placeholder="Select Masters/Equivalent"
              class="w-full mb-2 col-span-2"
              allowClear
              :options="mastersList"
              v-model:value="formData.education_info.masters.degree"
            ></a-select>
          </div>

          <div
            class="grid grid-cols-3"
            v-if="formData.education_info.masters.degree === 'Other'"
          >
            <label
              >Other Degree :
              <span class="text-red-500">*</span>
            </label>
            <a-input
              :required="requiredFields.masters"
              class="w-full mb-2 col-span-2"
              allowClear
              placeholder="Enter Other Degree..."
              v-model:value="other_degree_masters"
            />
          </div>

          <div class="grid grid-cols-3">
            <label
              >Subject/Department :
              <span class="text-red-500">*</span>
            </label>
            <a-select
              :required="requiredFields.masters"
              placeholder="Select Subject/Department"
              class="w-full mb-2 col-span-2"
              allowClear
              show-search
              :options="subjectList"
              v-model:value="formData.education_info.masters.group"
            ></a-select>
          </div>

          <div
            class="grid grid-cols-3"
            v-if="formData.education_info.masters.group === 'Other'"
          >
            <label
              >Other Subject :
              <span class="text-red-500">*</span>
            </label>
            <a-input
              :required="requiredFields.masters"
              class="w-full mb-2 col-span-2"
              allowClear
              placeholder="Enter other subject..."
              v-model:value="other_subject_masters"
            />
          </div>

          <div class="grid grid-cols-3">
            <label
              >University :
              <span class="text-red-500">*</span>
            </label>
            <a-select
              :required="requiredFields.masters"
              placeholder="Select University"
              class="w-full mb-2 col-span-2"
              allowClear
              show-search
              :options="institutionList"
              v-model:value="formData.education_info.masters.institute_name"
            ></a-select>
          </div>

          <div
            class="grid grid-cols-3"
            v-if="formData.education_info.masters.institute_name === 'Other'"
          >
            <label
              >Other University :
              <span class="text-red-500">*</span>
            </label>
            <a-input
              :required="requiredFields.masters"
              class="w-full mb-2 col-span-2"
              allowClear
              placeholder="Enter other University..."
              v-model:value="other_university_masters"
            />
          </div>

          <div class="grid grid-cols-3">
            <label
              >Result :
              <span class="text-red-500">*</span>
            </label>
            <a-select
              :required="requiredFields.masters"
              placeholder="Select Result"
              class="w-full mb-2 col-span-2"
              allowClear
              :options="bscGradeList"
              v-model:value="formData.education_info.masters.result"
            ></a-select>
          </div>

          <div
            class="grid grid-cols-3"
            v-if="
              ['CGPA(out of 4)', 'CGPA(out of 5)'].includes(
                formData.education_info.masters.result
              )
            "
          >
            <label
              >CGPA/GPA :
              <span class="text-red-500">*</span>
            </label>
            <a-input
              :required="requiredFields.masters"
              type="text"
              class="w-full mb-2 col-span-2"
              allowClear
              placeholder="CGPA/GPA"
              v-model:value="formData.education_info.masters.cgpa"
            />
          </div>

          <div class="grid grid-cols-3">
            <label
              >Passing Year :
              <span class="text-red-500">*</span>
            </label>
            <a-select
              :required="requiredFields.masters"
              placeholder="Select Passing Year"
              class="w-full mb-2 col-span-2"
              allowClear
              :options="yearList"
              v-model:value="formData.education_info.masters.passing_year"
            ></a-select>
          </div>

          <div class="grid grid-cols-3">
            <label>Achievements :</label>
            <a-input
              class="w-full mb-2 col-span-2"
              allowClear
              placeholder="Achievements (Academic, Scholarship, Distinctions, etc.)"
              v-model:value="formData.education_info.masters.achievements"
            />
          </div>
        </div>

        <!-- Below SSC One -->
        <div class="mb-8 education overflow-hidden">
          <label class="block mb-1"
            >Below SSC / Other Than Above (এসএসসির নিচে / উপরের ডিগ্রি ব্যতীত)
            (One)</label
          >

          <div class="grid grid-cols-3">
            <label
              >Degree :
              <!-- <span class="text-red-500">*</span> -->
            </label>
            <a-input
              class="w-full mb-2 col-span-2"
              allowClear
              placeholder="i.e JSC, JDC or Other"
              v-model:value="formData.education_info.otherone.degree"
            />
          </div>

          <div class="grid grid-cols-3">
            <label
              >Board :
              <!-- <span class="text-red-500">*</span> -->
            </label>
            <a-select
              placeholder="Select Board"
              class="w-full mb-2 col-span-2"
              allowClear
              :options="boardList"
              v-model:value="formData.education_info.otherone.board"
            ></a-select>
          </div>

          <div
            class="grid grid-cols-3"
            v-if="formData.education_info.otherone.board === 'Other'"
          >
            <label
              >Other Board :
              <!-- <span class="text-red-500">*</span> -->
            </label>
            <a-input
              class="w-full mb-2 col-span-2"
              allowClear
              placeholder="Enter Board..."
              v-model:value="formData.education_info.otherone.boardOther"
            />
          </div>

          <div class="grid grid-cols-3">
            <label
              >Group :
              <!-- <span class="text-red-500">*</span> -->
            </label>
            <a-input
              class="w-full mb-2 col-span-2"
              allowClear
              placeholder="Group"
              v-model:value="formData.education_info.otherone.group"
            />
          </div>

          <div class="grid grid-cols-3">
            <label
              >Institution/University :
              <!-- <span class="text-red-500">*</span> -->
            </label>
            <a-input
              class="w-full mb-2 col-span-2"
              allowClear
              placeholder="Institution/University"
              v-model:value="formData.education_info.otherone.institute_name"
            />
          </div>

          <div class="grid grid-cols-3">
            <label
              >Result :
              <!-- <span class="text-red-500">*</span> -->
            </label>
            <a-select
              placeholder="Select Result"
              class="w-full mb-2 col-span-2"
              allowClear
              :options="bscGradeList"
              v-model:value="formData.education_info.otherone.result"
            ></a-select>
          </div>

          <div
            class="grid grid-cols-3"
            v-if="
              ['CGPA(out of 4)', 'CGPA(out of 5)'].includes(
                formData.education_info.otherone.result
              )
            "
          >
            <label
              >CGPA/GPA :
              <!-- <span class="text-red-500">*</span> -->
            </label>
            <a-input
              type="text"
              class="w-full mb-2 col-span-2"
              allowClear
              placeholder="Enter CGPA/GPA"
              v-model:value="formData.education_info.otherone.cgpa"
            />
          </div>

          <div class="grid grid-cols-3">
            <label
              >Passing Year :
              <!-- <span class="text-red-500">*</span> -->
            </label>
            <a-select
              placeholder="Select Passing Year"
              class="w-full mb-2 col-span-2"
              allowClear
              :options="yearList"
              v-model:value="formData.education_info.otherone.passing_year"
            ></a-select>
          </div>

          <div class="grid grid-cols-3">
            <label>Achievements :</label>
            <a-input
              class="w-full mb-2 col-span-2"
              allowClear
              placeholder="Achievements"
              v-model:value="formData.education_info.otherone.achievements"
            />
          </div>
        </div>

        <!-- Below SSC two -->
        <div class="mb-8 education overflow-hidden">
          <label class="block mb-1"
            >Below SSC / Other Than Above (এসএসসির নিচে / উপরের ডিগ্রি ব্যতীত)
            (Two)</label
          >

          <div class="grid grid-cols-3">
            <label
              >Degree :
              <!-- <span class="text-red-500">*</span> -->
            </label>
            <a-input
              class="w-full mb-2 col-span-2"
              allowClear
              placeholder="i.e PSC or Other"
              v-model:value="formData.education_info.othertwo.degree"
            />
          </div>

          <div class="grid grid-cols-3">
            <label
              >Board :
              <!-- <span class="text-red-500">*</span> -->
            </label>
            <a-select
              placeholder="Select Board"
              class="w-full mb-2 col-span-2"
              allowClear
              :options="boardList"
              v-model:value="formData.education_info.othertwo.board"
            ></a-select>
          </div>

          <div
            class="grid grid-cols-3"
            v-if="formData.education_info.othertwo.board === 'Other'"
          >
            <label
              >Other Board :
              <!-- <span class="text-red-500">*</span> -->
            </label>
            <a-input
              class="w-full mb-2 col-span-2"
              allowClear
              placeholder="Enter Board..."
              v-model:value="formData.education_info.othertwo.boardOther"
            />
          </div>

          <div class="grid grid-cols-3">
            <label
              >Group :
              <!-- <span class="text-red-500">*</span> -->
            </label>
            <a-input
              class="w-full mb-2 col-span-2"
              allowClear
              placeholder="Group"
              v-model:value="formData.education_info.othertwo.group"
            />
          </div>

          <div class="grid grid-cols-3">
            <label
              >Institution/University :
              <!-- <span class="text-red-500">*</span> -->
            </label>
            <a-input
              class="w-full mb-2 col-span-2"
              allowClear
              placeholder="Institution/University"
              v-model:value="formData.education_info.othertwo.institute_name"
            />
          </div>

          <div class="grid grid-cols-3">
            <label
              >Result :
              <!-- <span class="text-red-500">*</span> -->
            </label>
            <a-select
              placeholder="Select Result"
              class="w-full mb-2 col-span-2"
              allowClear
              :options="bscGradeList"
              v-model:value="formData.education_info.othertwo.result"
            ></a-select>
          </div>

          <div
            class="grid grid-cols-3"
            v-if="
              ['CGPA(out of 4)', 'CGPA(out of 5)'].includes(
                formData.education_info.othertwo.result
              )
            "
          >
            <label
              >CGPA/GPA :
              <!-- <span class="text-red-500">*</span> -->
            </label>
            <a-input
              type="text"
              class="w-full mb-2 col-span-2"
              allowClear
              placeholder="Enter CGPA/GPA"
              v-model:value="formData.education_info.othertwo.cgpa"
            />
          </div>

          <div class="grid grid-cols-3">
            <label
              >Passing Year :
              <!-- <span class="text-red-500">*</span>  -->
            </label>
            <a-select
              placeholder="Select Passing Year"
              class="w-full mb-2 col-span-2"
              allowClear
              :options="yearList"
              v-model:value="formData.education_info.othertwo.passing_year"
            ></a-select>
          </div>

          <div class="grid grid-cols-3">
            <label>Achievements :</label>
            <a-input
              class="w-full mb-2 col-span-2"
              allowClear
              placeholder="Achievements"
              v-model:value="formData.education_info.othertwo.achievements"
            />
          </div>
        </div>

        <div class="text-right">
          <button
            type="submit"
            class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded w-32"
          >
            <span v-if="isLoading">Submiting...</span>
            <span v-else>Save & Next</span>
          </button>
        </div>
      </form>
    </fieldset>
  </MainLayout>
</template>

<style scoped>
.education {
  border: 1px solid #ddd;
  border-radius: 0.375rem;
  padding: 1rem;
  margin-bottom: 1.5rem;
}

.education h4 {
  margin-bottom: 1rem;
}

.education label {
  display: block;
  margin-bottom: 0.5rem;
}

.education .ant-select,
.education .ant-input {
  margin-bottom: 0.5rem;
}

.ant-select {
  width: 100%;
}

.ant-input {
  width: 100%;
}
</style>
