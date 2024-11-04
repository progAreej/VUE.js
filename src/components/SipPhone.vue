<template>
  <div class="min-h-screen bg-gradient-to-b from-gray-50 to-gray-100 flex items-center justify-center">
    <div class="w-full max-w-md bg-white rounded-2xl shadow-lg overflow-hidden">
      <div class="p-6 space-y-6">
        <!-- Connection Status -->
        <div class="flex items-center space-x-2">
          <div
            :class="`w-3 h-3 rounded-full transition-colors duration-300
              ${isConnected ? 'bg-green-500' : 'bg-red-500'}`"
          ></div>
          <span class="text-sm" :class="isConnected ? 'text-green-600' : 'text-red-600'">
            {{ isConnected ? 'Connected' : 'Disconnected' }}
          </span>
        </div>

        <!-- Dialer Interface -->
        <div class="space-y-4">
          <div class="relative">
            <input
              type="tel"
              v-model="destinationNumber"
              placeholder="Enter Phone Number"
              class="w-full px-4 py-3 border rounded-full focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none transition-all duration-300"
              :class="{ 'border-red-300': inputError }"
            />
            <span v-if="inputError" class="text-xs text-red-500 mt-1">
              {{ inputError }}
            </span>
          </div>

          <!-- Call Controls -->
          <div class="grid grid-cols-2 gap-4">
            <button
              @click="makeCall"
              :disabled="!isConnected || !destinationNumber"
              class="flex items-center justify-center px-4 py-3 rounded-full text-white font-medium transition-all duration-300"
              :class="{
                'bg-green-500 hover:bg-green-600': isConnected && destinationNumber,
                'bg-gray-400 cursor-not-allowed': !isConnected || !destinationNumber
              }"
            >
              <svg v-if="!isActive" class="w-6 h-6 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  d="M3 5a2 2 0 012-2h3.28a1 1 0 01.948.684l1.498 4.493a1 1 0 01-.502 1.21l-2.257 1.13a11.042 11.042 0 005.516 5.516l1.13-2.257a1 1 0 011.21-.502l4.493 1.498a1 1 0 01.684.949V19a2 2 0 01-2 2h-1C9.716 21 3 14.284 3 6V5z"
                />
              </svg>
              {{ isActive ? 'On Call' : 'Call' }}
            </button>

            <button
              @click="hangup"
              :disabled="!isActive"
              class="flex items-center justify-center px-4 py-3 rounded-full text-white font-medium transition-all duration-300"
              :class="{
                'bg-red-500 hover:bg-red-600': isActive,
                'bg-gray-400 cursor-not-allowed': !isActive
              }"
            >
              <svg class="w-6 h-6 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  d="M16 8l2-2m0 0l2-2m-2 2l-2-2m2 2l2 2M5 3a2 2 0 00-2 2v1c0 8.284 6.716 15 15 15h1a2 2 0 002-2v-3.28a1 1 0 00-.684-.948l-4.493-1.498a1 1 0 00-1.21.502l-1.13 2.257a11.042 11.042 0 01-5.516-5.517l2.257-1.13a1 1 0 00.502-1.21L9.228 3.683A1 1 0 008.279 3H5z"
                />
              </svg>
              Hang Up
            </button>
          </div>
        </div>

        <!-- Call Status -->
        <div v-if="status" class="p-4 rounded-2xl border transition-all duration-300" :class="{
          'bg-red-100 border-red-500 text-red-700': status.includes('failed'),
          'bg-green-100 border-green-500 text-green-700': status.includes('Connected'),
          'bg-blue-100 border-blue-500 text-blue-700': !status.includes('failed') && !status.includes('Connected')
        }">
          <div class="flex items-center space-x-2">
            <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"
              />
            </svg>
            <span>{{ status }}</span>
          </div>
        </div>

        <!-- Call Timer -->
        <div v-if="isActive" class="text-center">
          <span class="text-4xl font-mono text-gray-700">{{ callDuration }}</span>
        </div>

        <!-- Call History -->
        <transition-group name="fade" tag="div" class="bg-gray-100 p-4 rounded-2xl space-y-2">
          <div v-for="(call, index) in callHistory" :key="index" class="bg-white p-4 rounded-2xl shadow-md transition-all duration-300">
            <div class="flex justify-between items-center">
              <div>
                <span class="font-medium">Number:</span> {{ call.destination }}
              </div>
              <div>
                <span class="font-medium">Duration:</span> {{ call.duration }}
              </div>
            </div>
            <div class="mt-2 flex justify-between items-center">
              <span class="font-medium">Status:</span>
              <span
                :class="{
                  'text-red-500': call.status.includes('failed'),
                  'text-green-500': call.status.includes('Connected'),
                  'text-blue-500': !call.status.includes('failed') && !call.status.includes('Connected')
                }"
              >
                {{ call.status }}
              </span>
            </div>
          </div>
        </transition-group>
      </div>
    </div>



     <Transition name="modal">
      <div v-if="showIncomingCallModal" class="fixed inset-0 flex items-center justify-center z-50 bg-black bg-opacity-50">
        <div class="bg-white rounded-lg shadow-lg p-6 w-full max-w-md">
          <div class="mb-4">
            <h2 class="text-xl font-bold">Incoming Call</h2>
          </div>
          <div class="mb-4">
            <p>Incoming call from: {{ incomingCallInfo?.remoteIdentity }}</p>
          </div>
          <div class="flex justify-end">
            <button
              class="mr-2 px-4 py-2 bg-red-500 text-white rounded hover:bg-red-600"
              @click="rejectIncomingCall"
            >
              Reject
            </button>
            <button
              class="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600"
              @click="acceptIncomingCall"
            >
              Accept
            </button>
          </div>
        </div>
      </div>
    </Transition>
  </div>
</template>

<script>
import { UserAgent, Registerer, Inviter } from 'sip.js';
import { ref, onMounted, onUnmounted } from 'vue';


export default {
  name: 'SIPClient',
  
  data() {
    return {
      userAgent: null,
      registerer: null,
      session: null,
      status: '',
      isActive: false,
      isConnected: false,
      destinationNumber: '',
      callStartTime: null,
      callDuration: '00:00',
      durationInterval: null,
      debugInfo: 'Initializing...',
      showDebug: false,
      inputError: '',
      mediaStream: null,
      audioElement: null,
            callSummary: null,
                  callHistory: [],
               showIncomingCallModal: false,
     incomingCallInfo: null



    };
  },

  async created() {
    await this.initializeSIP();
    // Create audio element for call audio
    this.audioElement = document.createElement('audio');
    this.audioElement.autoplay = true;
  },

  beforeUnmount() {
    this.cleanup();
  },

  methods: {
    async initializeSIP() {
      try {
        const uri = UserAgent.makeURI('sip:101@sip.xtrevo.com');
        if (!uri) throw new Error('Failed to create URI');

        this.userAgent = new UserAgent({
          uri,
          authorizationUsername: '101',
          authorizationPassword: 'severalleddepend8912@@',
          transportOptions: {
            server: 'wss://sip.xtrevo.com:8443/asterisk-wss'
          }
        });

        this.registerer = new Registerer(this.userAgent);

        this.userAgent.transport.onConnect = () => {
          this.debugInfo = 'Transport connected';
        };

        this.userAgent.transport.onDisconnect = (error) => {
          this.debugInfo = `Transport disconnected: ${error}`;
          this.isConnected = false;
        };

        this.registerer.stateChange.addListener((state) => {
          this.debugInfo = `Registration state: ${state}`;
          if (state === 'Registered') {
            this.isConnected = true;
          }
        });

        this.userAgent.delegate = {
          onInvite: (invitation) => {
            this.handleIncomingCall(invitation);
          }
        };

        await this.userAgent.start();
        await this.registerer.register();
        
        this.debugInfo = 'SIP client initialized successfully';
        this.isConnected = true;
      } catch (error) {
        this.debugInfo = `Failed to initialize SIP client: ${error}`;
        console.error('Failed to initialize SIP client:', error);
        this.status = 'Failed to connect to SIP server';
        this.isConnected = false;
      }
    },

    validateDestinationNumber() {
      if (!this.destinationNumber) {
        this.inputError = 'Please enter a destination number';
        return false;
      }
      if (!/^\d+$/.test(this.destinationNumber)) {
        this.inputError = 'Please enter only numbers';
        return false;
      }
      this.inputError = '';
      return true;
    },

    async makeCall() {
      if (!this.validateDestinationNumber() || !this.isConnected) return;

      try {
        const targetUri = UserAgent.makeURI(`sip:${this.destinationNumber}@sip.xtrevo.com`);
        if (!targetUri) throw new Error('Failed to create target URI');

        this.debugInfo = `Creating invitation to ${targetUri}`;

        const inviter = new Inviter(this.userAgent, targetUri, {
          sessionDescriptionHandlerOptions: {
            constraints: {
              audio: true,
              video: false
            }
          }
        });

        // Set up session state change handler
        this.setupSessionHandlers(inviter);

        await inviter.invite();
        this.session = inviter;
        this.status = 'Calling...';
      } catch (error) {
        this.debugInfo = `Call failed: ${error}`;
        console.error('Call failed:', error);
        this.status = 'Call failed';
        this.isActive = false;
      }
    },

    async hangup() {
      if (this.session) {
        try {
          if (this.session instanceof Inviter) {
            await this.session.bye();
          } else {
            // For incoming calls (Invitation)
            await this.session.bye();
          }
          this.status = 'Call ended';
          this.cleanup();
        } catch (error) {
          this.debugInfo = `Error ending call: ${error}`;
          console.error('Error ending call:', error);
          this.status = 'Error ending call';
        }
      }
    },

    setupSessionHandlers(session) {
      session.stateChange.addListener((state) => {
        this.debugInfo = `Call state changed to: ${state}`;
        switch (state) {
          case 'Establishing':
            this.status = 'Calling...';
            break;
          case 'Established':
            this.status = 'Connected';
            this.isActive = true;
            this.startTimer();
            // Handle media stream
            if (session.sessionDescriptionHandler?.peerConnection) {
              const pc = session.sessionDescriptionHandler.peerConnection;
              pc.getReceivers().forEach(receiver => {
                if (receiver.track) {
                  this.mediaStream = new MediaStream([receiver.track]);
                  this.audioElement.srcObject = this.mediaStream;
                }
              });
            }
            break;
          case 'Terminated':
            this.status = 'Call ended';
            this.cleanup();
            break;
        }
      });
    },

    async handleIncomingCall(invitation) {
      this.debugInfo = `Incoming call from: ${invitation.remoteIdentity.uri.user}`;
      
      if (confirm('Incoming call. Do you want to accept?')) {
        try {
          this.session = invitation;
          
          // Set up session state change handler
          this.setupSessionHandlers(invitation);

          await invitation.accept({
            sessionDescriptionHandlerOptions: {
              constraints: {
                audio: true,
                video: false
              }
            }
          });
          
          this.status = 'Call connected';
          this.isActive = true;
          this.startTimer();
        } catch (error) {
          this.debugInfo = `Error accepting call: ${error}`;
          console.error('Error accepting call:', error);
          this.status = 'Error accepting call';
        }
      } else {
        try {
          await invitation.reject();
          this.status = 'Call rejected';
        } catch (error) {
          this.debugInfo = `Error rejecting call: ${error}`;
          console.error('Error rejecting call:', error);
        }
      }
    },

    startTimer() {
      this.callStartTime = new Date();
      this.durationInterval = setInterval(() => {
        const duration = Math.floor((new Date() - this.callStartTime) / 1000);
        const minutes = Math.floor(duration / 60).toString().padStart(2, '0');
        const seconds = (duration % 60).toString().padStart(2, '0');
        this.callDuration = `${minutes}:${seconds}`;
      }, 1000);
    },

    cleanup() {
      // Clear timer
      if (this.durationInterval) {
        clearInterval(this.durationInterval);
        this.durationInterval = null;
      }

      // Clean up media
      if (this.audioElement) {
        this.audioElement.srcObject = null;
      }
      if (this.mediaStream) {
        this.mediaStream.getTracks().forEach(track => track.stop());
        this.mediaStream = null;
      }

      this.isActive = false;
      this.session = null;
      this.callStartTime = null;
      this.callDuration = '00:00';
     this.callHistory.push({
        destination: this.destinationNumber,
        duration: this.callDuration,
        status: this.status
      });
    
    
    }
  }
};
</script>

<style scoped>
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.3s ease;
}
.fade-enter-from, .fade-leave-to {
  opacity: 0;
}
</style>