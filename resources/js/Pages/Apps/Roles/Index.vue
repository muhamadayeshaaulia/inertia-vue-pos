<template>
  <Head>
    <title>Roles - CAFE-MAMI</title>
    <link rel="icon" href="/images/vue2.png" type="image/png" />
  </Head>

  <main class="c-main">
    <div class="container-fluid">
      <div class="fade-in">
        <div class="row">
          <div class="col-md-12">
            <div class="card border-0 rounded-3 shadow border-top-purple">
              <div class="card-header">
                <span class="font-weight-bold">
                  <i class="fa fa-shield-alt"></i> ROLES
                </span>
              </div>
              <div class="card-body">
                <form @submit.prevent="handleSearch">
                  <div class="input-group mb-3">
                    <Link
                      href="/apps/roles/create"
                      v-if="hasAnyPermission(['roles.create'])"
                      class="btn btn-primary input-group-text"
                    >
                      <i class="fa fa-plus-circle me-2"></i> NEW
                    </Link>
                    <input
                      type="text"
                      class="form-control"
                      v-model="search"
                      placeholder="search by role name..."
                    />
                    <button class="btn btn-primary input-group-text" type="submit">
                      <i class="fa fa-search me-2"></i> SEARCH
                    </button>
                  </div>
                </form>

                <!-- Scrollable Table with Buttons -->
                <div class="table-scroll-wrapper">
                  <button class="scroll-btn left" @click="scrollTable('left')">&#8592;</button>

                  <div class="table-responsive" ref="tableScroll">
                    <table class="table table-striped table-bordered table-hover">
                      <thead>
                        <tr>
                          <th scope="col">Role Name</th>
                          <th scope="col" style="width:50%">Permissions</th>
                          <th scope="col" style="width:20%">Actions</th>
                        </tr>
                      </thead>
                      <tbody>
                        <tr v-for="(role, index) in roles.data" :key="index">
                          <td>{{ role.name }}</td>
                          <td>
                            <span
                              v-for="(permission, index) in role.permissions"
                              :key="index"
                              class="badge badge-primary shadow border-0 ms-2 mb-2"
                            >
                              {{ permission.name }}
                            </span>
                          </td>
                          <td class="text-center">
                            <Link
                              :href="`/apps/roles/${role.id}/edit`"
                              v-if="hasAnyPermission(['roles.edit'])"
                              class="btn btn-success btn-sm me-2"
                            >
                              <i class="fa fa-pencil-alt me-1"></i> EDIT
                            </Link>
                            <button
                              @click.prevent="destroy(role.id)"
                              v-if="hasAnyPermission(['roles.delete'])"
                              class="btn btn-danger btn-sm"
                            >
                              <i class="fa fa-trash"></i> DELETE
                            </button>
                          </td>
                        </tr>
                      </tbody>
                    </table>
                  </div>

                  <button class="scroll-btn right" @click="scrollTable('right')">&#8594;</button>
                </div>

                <Pagination :links="roles.links" align="end" />
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </main>
</template>

<script>
import LayoutApp from "../../../layouts/App.vue";
import Pagination from "../../../Components/Pagination.vue";
import { Head, Link, router } from "@inertiajs/vue3";
import { ref, watch } from "vue";
import Swal from "sweetalert2";

export default {
  layout: LayoutApp,
  components: {
    Head,
    Link,
    Pagination,
  },
  props: {
    roles: Object,
  },
  setup() {
    const search = ref((new URL(document.location)).searchParams.get("q") || "");
    const tableScroll = ref(null);

    const handleSearch = () => {
      router.get("/apps/roles", {
        q: search.value,
      });
    };

    const destroy = (id) => {
      Swal.fire({
        title: "Are you sure?",
        text: "You won't be able to revert this!",
        icon: "warning",
        showCancelButton: true,
        confirmButtonColor: "#3085d6",
        cancelButtonColor: "#d33",
        confirmButtonText: "Yes, delete it!",
      }).then((result) => {
        if (result.isConfirmed) {
          router.delete(`/apps/roles/${id}`);
          Swal.fire({
            title: "Deleted!",
            text: "Role deleted successfully.",
            icon: "success",
            timer: 2000,
            showConfirmButton: false,
          });
        }
      });
    };

    const scrollTable = (direction) => {
      const el = tableScroll.value;
      if (!el) return;
      const amount = 200;
      el.scrollLeft += direction === "right" ? amount : -amount;
    };

    watch(search, (newVal) => {
      if (newVal === "") {
        router.get("/apps/roles", {}, {
          preserveState: true,
          preserveScroll: true,
        });
      }
    });

    return {
      search,
      handleSearch,
      destroy,
      tableScroll,
      scrollTable,
    };
  },
};
</script>

<style>
.table-scroll-wrapper {
  display: flex;
  align-items: center;
  overflow: hidden;
  margin-bottom: 1rem;
}

.table-responsive {
  width: 100%;
  overflow-x: auto;
  -webkit-overflow-scrolling: touch;
  scroll-behavior: smooth;
  flex: 1;
}

.table-responsive table {
  min-width: 768px;
}

.scroll-btn {
  background: #42b883;
  color: white;
  border: none;
  padding: 6px 10px;
  font-size: 1.2rem;
  cursor: pointer;
  z-index: 2;
  border-radius: 4px;
}

@media (min-width: 768px) {
  .scroll-btn {
    display: none;
  }
}
</style>
