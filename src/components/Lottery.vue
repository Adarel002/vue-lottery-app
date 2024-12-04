<script setup lang="ts">
import WinnersList from "@/components/WinnersList.vue";
import UserForm from "@/components/UserForm.vue";
import UserList from "@/components/UserList.vue";
import { onMounted, ref } from "vue";
import type { CreateUserRequest, User } from "@/models/user";
import { LocalRepository } from "@/helpers/local-repository";
import axios from "axios";

const users = ref<User[]>([]);
const winners = ref<User[]>([]);
const usersKey: string = "vue-lottery-app.users";
const buttonRef = ref<HTMLButtonElement | null>(null);
const showModal = ref(false); 

onMounted(async () => {
  try {
    const response = await axios.get('https://jsonplaceholder.typicode.com/users');
    users.value = response.data.map((user: any) => ({
      id: user.id,
      name: user.name,
      email: user.email,
      role: user.role || 'customer',
      avatar: user.avatar || '', 
    }));
  } catch (error) {
    console.error("Error fetching users:", error);
  }
});

const updateUsersInStorage = () => {
  LocalRepository.set(usersKey, users.value);
};

const deleteUser = (user: User) => {
  users.value = users.value.filter(u => u !== user);
  updateUsersInStorage();
};

const getNextId = () => {
  if (users.value.length === 0) {
    return 1;
  }

  const maxId = Math.max(...users.value.map(user => user.id));
  
  return maxId + 1;
};

const addUser = async (user: CreateUserRequest) => {
  console.log(user);
  if (users.value.filter(u => u.email === user.email).length > 0) {
    showModal.value = true;
    return;
  }
  const userWithId = { ...user, id: getNextId() };
  const response = await axios.post('/users', userWithId);
  users.value.push({ ...response.data });
  updateUsersInStorage();
};

onMounted(async () => {
  const storedUsers = LocalRepository.get(usersKey);
  if (storedUsers) {
    users.value = storedUsers;
  } else {
    try {
      const response = await axios.get('/users');
      users.value = response.data.map((user: any) => ({
        id: user.id,
        name: user.name,
        email: user.email,
        role: user.role || 'customer',
        avatar: user.avatar || '', 
      }));
    } catch (error) {
      console.error("Error fetching users:", error);
    }
  }
});

const deleteWinner = (user: User) => {
  winners.value = winners.value.filter((x) => x.email !== user.email);
};

const addNewWinner = () => {
  const randomIndex = Math.floor(Math.random() * users.value.length);
  const winner = users.value[randomIndex];
  if (winners.value.filter((w) => w.email === winner.email).length > 0) {
    return;
  }
  winners.value.push(winner);
};

const updateUser = async (user: User) => {
  users.value = users.value.filter(u => u.email !== user.email);
  users.value.push(user);
  await axios.put(`/users/${user.id}`, user);
  updateUsersInStorage();
};
</script>

<template>
  <div class="w-75 m-auto">
    <WinnersList
      :users-count="users.length"
      @deleteWinner="deleteWinner"
      :winners="winners"
      @newWinner="addNewWinner"
    ></WinnersList>
    <UserForm @addUser="addUser"></UserForm>
    <UserList @updateUser="updateUser" @deleteUser="deleteUser" :users="users"></UserList>
  </div>

  <div v-if="showModal" class="modal fade show d-block" id="exampleModal" tabindex="-1" aria-labelledby="exampleModalLabel" aria-hidden="true">
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header">
          <h1 class="modal-title fs-5" id="exampleModalLabel">Conflict while adding new user</h1>
          <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close" @click="showModal = false"></button>
        </div>
        <div class="modal-body">
          User with given email already exists
        </div>
        <div class="modal-footer">
          <button type="button" class="btn btn-primary" data-bs-dismiss="modal" @click="showModal = false">Ok</button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
table {
  border-collapse: collapse;
  width: 100%;
}

th, td {
  padding: 12px;
  text-align: left;
  border-bottom: 1px solid #ddd;
}

th {
  background-color: #f2f2f2;
}

.avatar {
  width: 30px;
  height: 30px;
  border-radius: 50%;
}
</style>
