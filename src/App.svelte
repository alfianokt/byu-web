<!-- 
	byu-web
	cek kuota byu dari web

	author	: alfianokt
	date		: 4-5 03 2020
	git			: https://github.com/alfianokt/byu-web
`	demo		: https://byu.surge.sh
 -->
<script>
	import Cookies from 'js-cookie'
	import NProgress from 'nprogress'
	import { writable } from 'svelte/store'
	import { onMount } from 'svelte'
	let savedToken = null
	let tempToken = null
	let data = {
		show: false,
		data: {  }
	}
	let msg = {
		show: false,
		data: null
	}

	/**
	 * onMount
	 * 
	 * this function will be call after page created
	 */
	onMount(() => {
		if(Cookies.get('byuToken') != undefined) {
			savedToken = Cookies.get('byuToken')
			byu()
		}
	})

	/**
	 * alert
	 * 
	 * show alert to user
	 * 
	 * @param data String
	 */
	const showAlert = (data) => {
		msg = {
			show: true,
			data
		}

		setTimeout(() => {
			hideAlert()
		}, 3000);
	}

	/**
	 * hideAlert
	 * 
	 * hide alert message
	 */
	const hideAlert = () => {
		msg = {
			show: false,
			data: null
		}
	}
	/**
	 * request
	 * 
	 * HTTP request to call api
	 * 
	 * @param token String default savedToken 
	 */
	const request = async (token=savedToken) => {
		try {
			NProgress.start()
			data = {
				show: false,
				data: {  }
			}
			const req = await fetch('https://api.byu.id/api/planRemaining', {
				headers: {
					'Authorization':'Bearer ' + token
				}
			})
			NProgress.done();
			return await req.json()
		} catch (e) {
			throw new Error('Failed to fetch API.')
		}
	}

	/**
	 * save
	 * 
	 * validate and save token to cookie
	 */
	const save = async () => {
		if (tempToken != null) {
			const req = await request(tempToken)
			// if token error
			if (req.status != undefined) {
				showAlert('Error : ' + req.message)
			} else {
				Cookies.set('byuToken', tempToken)
				savedToken = tempToken
				byu()
			}
		} else {
			showAlert('Token masih kosong!')
		}
	}
	/**
	 * byu
	 * 
	 * show data to table
	 */
	const byu = async () => {
		try {
			const req = await request()
			// if error
			if(req.status != undefined) {
				showAlert('Error : ' + req.message)
				// not recomended in slow network
				// logout()
			}
			data = {
				show: true,
				data: req
			}
		} catch (e) {
			showAlert(e)
		}
	}

	/**
	 * Logout
	 * 
	 * remove cookie and give null value to savedToken and tempToken 
	 */
	const logout = () => {
		Cookies.remove('byuToken')
		savedToken = null
		tempToken = null
	}

	/**
	 * tokenAvailable
	 * 
	 * reactive function to check cookie and savedToken
	 */ 
	$: tokenAvailable = function () {
		return Cookies.get('byuToken') != undefined || savedToken != null
	}
</script>


<style>
	.container {
		height: 90vh;
	}

	footer {
		height: 10vh;
		display: grid;
		justify-items: center;
		align-items: center;
	}
</style>

<main>
	<div class="container">
		<div class="columns">
			<div class="column col-sm-12 col-md-8 col-6 col-mx-auto">
				<div class="card mt-2">
					<div class="card-header">
						<div class="card-title h5">
							Byu
							{#if tokenAvailable()}
								<div class="float-right">
									<button class="btn" on:click={logout}>Keluar <i class="icon icon-shutdown"></i></button>
								</div>
							{/if}
						</div>
						<p class="card-description text-gray">Cek kuota byu via web.</p>
					</div>
					<div class="card-body">
						<!-- warn toast -->
						{#if msg.show}
							<div class="toast toast-warning mb-2">
								<button class="btn btn-clear float-right" on:click={hideAlert}></button>
								{msg.data}
							</div>
						{/if}
						{#if tokenAvailable()}
							<button class="btn" on:click={byu}>Refresh <i class="icon icon-refresh"></i></button>
							<table class="table table-striped table-hover">
								<thead>
									<tr>
										<th>#</th>
										<th>Sisa</th>
									</tr>
								</thead>
								<tbody>
									<tr>
										<td>Pulsa</td>
										<td>
											{#if data.show}
												{data.data.total.credit.total_offering} {data.data.total.credit.unit_id}
											{:else}
												<progress class="progress" max="100"></progress>
											{/if}
										</td>
									</tr>
									<tr>
										<td>Kuota</td>
										<td>
										{#if data.show}
											{data.data.total.data_plan.remaining} {data.data.total.data_plan.unit_id}
										{:else}
											<progress class="progress" max="100"></progress>
										{/if}
										</td>
									</tr>
									<tr>
										<td>Kuota Tambahan</td>
										<td>
											{#if data.show}
												{data.data.total.data_addon.remaining} {data.data.total.data_addon.unit_id}
											{:else}
												<progress class="progress" max="100"></progress>
											{/if}
										</td>
									</tr>
									<tr>
										<td>Telpon</td>
										<td>
											{#if data.show}
												{data.data.total.voice.total_offering} {data.data.total.voice.unit_id}
											{:else}
												<progress class="progress" max="100"></progress>
											{/if}
										</td>
									</tr>
								</tbody>
							</table>
						{:else}
							<div class="input-group">
								<input type="text" class="form-input" placeholder="Masukkan token.." bind:value={tempToken}>
								<button class="btn btn-primary input-group-btn" on:click={save} class:disabled={tempToken == null}>Masuk <i class="icon icon-forward"></i></button>
							</div>
						{/if}
					</div>
				</div>
			</div>
		</div>
	</div>
	<footer class="bg-gray">
		<div class="text-center">
			Created with <span class="text-error">&hearts;</span> by <a href="https://github.com/alfianokt" target="_blank" rel="">Alfian Oktafireza</a>
		</div>
	</footer>
</main>