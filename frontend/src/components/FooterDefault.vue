<template>
    <footer>
        <p>&copy; All rights reserved</p>
    </footer>
</template>

<style scoped>
@import '../assets/variables.css';

footer {
    background-color: var(--color-primary);
    color: var(--color-on-primary);
    display: flex;
    justify-content: center;
    align-items: center;
    flex-shrink: 0;
    padding-block: 25px;

    @media (max-width: 767px) {
        padding-block: 15px;
    }
}
</style>