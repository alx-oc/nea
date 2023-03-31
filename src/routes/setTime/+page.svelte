<div class="container">
    <a href="./chess?time=60">1 minute</a>
    <a href="./chess?time=180">3 minutes</a>
    <a href="./chess?time=300">5 minutes</a>
    <a href="./chess?time=600">10 minutes</a>
</div>

<style>
    @import url('https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&display=swap');

    .container {
        display: flex;
        justify-content: space-between;
        max-width: 800px;
        margin: 0 auto;
        margin-top: 50px;
    } a {
        flex: 1;
        margin: 0 10px;
        padding: 10px 20px;
        font-family: 'Roboto', sans-serif;
        font-size: 1.2em;
        font-weight: 700;
        background-color: #0077b6;
        color: #fff;
        border: none;
        border-radius: 5px;
        cursor: pointer;
    } a:hover {
        background-color: #023e8a;
    }
</style>